Zan Thrift IDL 设计规范 
=========================

thrift IDL项目 目录结构
-----------------------

::

    ├── composer.json
    ├── gen_java.sh # Java SDK生成脚本
    ├── gen_php.sh # PHP SDK 生成脚本
    ├── gen-php # PHP SDK 生成目录
    ├── thrifts # IDL目录
    │   └── credit # 应用的领域,具体模块名称
    │       └── base # 分层, core, base
    │           ├── common # 这个应用的领域的分层内的通用实体和业务异常
    │           │   ├── entity
    │           │   └── exception
    │           ├── policy # 子领域
    │           │   ├── constant # 常量定义
    │           │   ├── entity # 实体定义
    │           │   ├── exception # 业务异常
    │           │   └── service # 服务定义

Service IDL 模板
----------------

service的方法定义, 需要采用 /\*\* \*/ 将注释文本包围,
并且在每个参数的序号前, 使用这个格式给出这个参数的注释,
对于异常也需要用同样的方式给出

.. code:: thrift

    namespace nova com.youzan.scrm.credit.base.service
    /**
     * 首行是命名空间定义, 命名空间与目录名是保持一致的
     * com.youzan.scrm 为SCRM的固定前缀，credit 为领域名称
     */

    /**
     * 引入接口用到的 DTO 定义
     * 以相对于 thrifts 目录引入 DTO 文件，IDE 可以识别并高亮
     */
    include "credit/base/entity/account/CreditAccountDTO.thrift"
    include "credit/base/entity/account/CreditAccountModificationDTO.thrift"
    include "credit/base/entity/account/CreditAccountQueryDTO.thrift"
    include "credit/base/exception/common/CreditBaseException.thrift"
    include "credit/base/exception/common/NegativeCreditException.thrift"

    service CreditAccountService {

        /**
         * 增加积分
         */
        CreditAccountModifiedResultDTO.CreditAccountModifiedResultDTO increaseCredit(
            /** 账户查询参数 */
            1:CreditAccountQueryDTO.CreditAccountQueryDTO accountQuery,
            /** 积分变动对象 */
            2:CreditAccountModificationDTO.CreditAccountModificationDTO modification
        ) throws (
            /** 未定义积分异常 */
            1:CreditBaseException.CreditBaseException e
        );

        /**
         *  消耗积分
         */
        CreditAccountModifiedResultDTO.CreditAccountModifiedResultDTO decreaseCredit(
            /** 账户查询参数 */
            1:CreditAccountQueryDTO.CreditAccountQueryDTO accountQuery,
            /** 积分变动对象 */
            2:CreditAccountModificationDTO.CreditAccountModificationDTO modification
        ) throws (
            /** 未定义积分异常 */
            1:CreditBaseException.CreditBaseException e,
            /** 积分不可为负异常 */
            2:NegativeCreditException.NegativeCreditException negativeE
        );
    }

1. 尽量以 CRUD的顺序定义接口，保证同类型的接口存放位置是相邻的，便于查找

2. 接口

   1. 动宾结构
   2. 表意，如 getByCardAlias
   3. 通常用 getBrief 表示获取简要信息
   4. 所有接口都必须定义抛出的异常类型

3. 返回值

   1. 一般情况下，create 接口返回创建的对象的完整 DTO，update, delete之类的操作接口返回布尔值
   2. 列表接口，若需要返回分页信息，返回值 DTO 名称使用 XXXPaginationDTO格式
   3. 根据 ID 获取列表，返回 map

4. 参数

   1. 接口定义要明确操作主体，一般地操作主体的参数都放在第一位，如 kdtId
   2. 接口一旦有使用方使用了，不得随意调整参数的顺序值
   3. 参数值若有限，必须使用枚举类型定义
   4. 参数值为不可重复数值列表（如 id 列表），请使用 set 类型
   5. 参数值为数字，请酌情使用 i16 i32 i64
   6. 带复杂查询条件、排序条件、分页要求的接口参数建议：
      
      .. code:: thrift

        1:i32 kdtId,
        2:CardQueryDTO.CardQueryDTO query, 
        3:map order, 
        4:i32 page,
        5:i32 pageSize

      1. 查询条件命名成 XXXQueryDTO
      2. 排序条件定义成 map，key 为排序字段，value 为 DESC or ASC

   7. 不要使用 json 字符串传值，请使用类型组合的方式
   8. 入参与返回值使用的 DTO 请分别定义，通常入参的 DTO 会命名成
      XXXCreateDTO, XXXUpdateDTO

DTO IDL 模板
------------

model的每个字段, 注释也是使用 /\*\* \*/ 包围,
并且每个属性都需要写出是必须还是可选

.. code:: thrift

    namespace nova com.youzan.scrm.credit.base.account.entity

    include "credit/base/common/entity/CreditCycleDTO.thrift"
    include "credit/base/common/entity/CreditStatusDTO.thrift"
    include "credit/base/source/entity/SourceBusinessQueryDTO.thrift"
    include "credit/base/source/entity/SourceUserQueryDTO.thrift"

    /**
     * 用于描述一个用户, 在一个商家的一个积分定义以及一个积分策略下的积分概览信息
     */
    struct CreditAccountDTO {

        /** 账户ID */
        1:required i64 accountId;

        /**
         * 外部商家描述
         */
        2:required SourceBusinessQueryDTO.SourceBusinessQueryDTO business;

        /**
         * 外部用户描述
         */
        3:required SourceUserQueryDTO.SourceUserQueryDTO user;

        /**
         * 积分定义ID
         */
        6:required i64 definitionId;

        /**
         * 策略ID
         */
        7:required i64 policyId;

        /**
         * 当前积分状态
         */
        101:required CreditStatusDTO.CreditStatusDTO status;

        /**
         * 下一次将要过期的积分的信息
         */
        102:optional CreditCycleDTO.CreditCycleDTO willExpire;

        /**
         * 最后一次过期的积分的信息
         */
        103:optional CreditCycleDTO.CreditCycleDTO lastExpired;

        /**
         * 创建积分账号的时间, UNIX时间戳, 以秒计
         */
        201:optional i64 createdAt = 0;

        /**
         * 积分账号更新的时间, UNIX时间戳, 以秒计
         */
        202:optional i64 updatedAt = 0;
    }

1. 必须显式的声明字段是required和optional, 其中

   1. required不允许有默认值
   2. optional可以有默认值

2. 请根据实际情况使用各种数据类型，特别是 map, set, list
   以及自定义的枚举类型
3. 字段的顺序值不得随意调整, 并建议进行一定的分段以容纳业务上的新字段
   (见上面例子, 7和101中间分开了很多空余,
   足够给未来添加新的字段使用并且可以维持一定的逻辑顺序)
4. 几种特殊约定的 DTO 命名，用途在接口定义一节已说明

   1. XXXCreateDTO

      1. 不会包含 kdtId, uid 等主体参数

   2. XXXUpdateDTO

      1. 必须包含 kdtId, uid 等主体参数

   3. XXXQueryDTO
   4. XXXListItemDTO
   5. XXXPaginationDTO

.. code:: thrift

    struct XXXPaginationDTO {
        1:i64 total;
        2:i32 page;
        3:i32 pageSize;
        4:list<XXXListItemDTO.XXXListItemDTO> items;
    }

Exception IDL 模板
------------------

Exception没必要带上对于属性的注释,
但是强制要求message和code两个属性直接给出默认值

.. code:: thrift

    namespace nova com.youzan.scrm.credit.base.exception.common

    exception NegativeCreditException {

        1:optional string message = '积分不能为负'

        2:optional i32 code = 140000000

    }
