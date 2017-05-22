# 异常处理


> 当一个异常没有在业务逻辑里捕获处理的情况下，框架会接管异常并进行处理。

### ZanHttp异常处理流程

1. 如果exception有redirectUrl并且instance of RedirectException，将会跳转至指定页面。
2. 如果异常类型是PageNotFoundException，则会跳转配置的404页面。
3. 如果异常类型是InvalidRouteException，则会跳转到配置的404页面。
4. 如果是exception code在业务异常定义范围内，则会根据request类型选择输出json或者html错误信息
5. 如果上述异常均未匹配，如果有配置的500页面则跳转，否则输出霸王龙页面。

> 以上流程从1~5按顺序逐个匹配，当匹配上某一条则后面的都不执行，如果都没有匹配上会走5的逻辑。


当然，也可以在项目中通过配置文件`exception.php`来增加异常类到框架异常调用链的顶端，优先处理。如下：

`resource/share/exception.php`

```
return array(
	  //handles里面的内容会追加到框架异常调用链的最顶部
    'handles' => array(
        \Store\Common\Exception\StoreExceptionHandle::class
    )
);
```

`Common/Exception/StoreExceptionHandle.php`

```
namespace Store\Common\Exception;

use Zan\Framework\Contract\Foundation\ExceptionHandler;
use Zan\Framework\Foundation\Core\Path;
use Zan\Framework\Foundation\Core\RunMode;
use Zan\Framework\Network\Http\Response\JsonResponse;
use Zan\Framework\Network\Http\Response\Response;

class StoreException implements ExceptionHandler {

	public function handle(\Exception $e) {
  		//这里写自己的异常处理逻辑，会优先处理。。。  
	}
```


### Debug模式下

如果在debug模式下，在执行上述流程之外，还会在STDOUT中打印异常信息，方便调试。
