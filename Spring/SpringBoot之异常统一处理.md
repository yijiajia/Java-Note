## 统一处理返回结果

> 当后台在开发过程中，往往需要返回一个json对象给前端。当出现异常时，我们同样希望能把异常按照json格式进行返回，前端就可以根据返回json数据的状态码和信息进行相应的显示。

这时候就需要写一个Http最外层的封装对象，统一返回的对象数据。
Result.java

```
public class Result<T> {

    private Integer code;
	private String msg;
	private T data;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```

根据返回数据对象可封装相应的结果模板工具类
ResultUtil.java
```
public class ResultUtil {

    public static Result getOK(Object object){
        Result result = new Result();
        result.setCode(0);
        result.setMsg("成功");
        result.setData(object);
        return result;
    }

    public static Result getOK(){
        return getOK(null);
    }

    public static Result getError(Integer code,String msg){
        Result result = new Result();
        result.setCode(code);
        result.setMsg(msg);
        return result;
    }
}
```

当后台对前端传过来的数据进行判断，返回相应的结果时，可以进行统一异常处理。
将结果抛出，如果直接进行抛出的话，结果并不是之前我们返回的json格式，这样的处理也不友好。我们可以通过对异常的捕获，调用结果模板类（ResultUtil.java）即可返回我们封装好的json数据。

异常捕获类ExceptionHandle.java
```java
import com.demo.result.Result;
import com.demo.result.ResultUtil;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@ControllerAdvice
public class ExceptionHandler {
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Result handle(Exception e){
        return ResultUtil.getError(100,e.getMessage());
    }

}
```


如果只是利用默认Exception进行抛出结果，这样返回的状态码(code)每次都是同一个值，前端处理的时候就不好处理。这样就需要自定义一个Exception。注意这里不能继承Exception，因为 springBoot只支持继承RuntimeException的。

自定义Exception类UserException.java
```java
public class UserException extends RuntimeException {

    private Integer code;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public UserException(Integer code,String message) {
        super(message);
        this.code = code;
    }
}
```
此时异常捕获类ExceptionHandle.java也需要更改
```java
import com.demo.exception.UserException;
import com.demo.result.Result;
import com.demo.result.ResultUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@ControllerAdvice
public class ExceptionHandle {
    
    //增加异常日志打印
    private final static Logger logger = LoggerFactory
										    .getLogger(ExceptionHandle.class);

    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Result handle(Exception e){
        if(e instanceof UserException){
            UserException userException = (UserException)e;
            return ResultUtil.getError(userException.getCode()
									           ,userException.getMessage());
        }else{
            logger.error("【系统异常】={}",e);
            return ResultUtil.getError(-1,"未知错误！");
        }
    }

}
```



通过对结果的统一处理，可以很友好的将数据返回给前端，但此刻发现一个问题，就是每次返回数据时，状态码和信息都要在调用方法中重新定义，这样做的话状态多一点，查看和修改就有了困难。为此可定义一个枚举类，统一管理code和msg。
ResultEnum.java

```java
public enum ResultEnum {
    /**
     * 成功. ErrorCode : 0
     */
    SUCCESS("0","成功"),
    /**
     * 未知异常. ErrorCode : 01
     */
    UnknownException("01","未知异常"),
    /**
     * 系统异常. ErrorCode : 02
     */
    SystemException("02","系统异常"),
    /**
     * 业务错误. ErrorCode : 03
     */
    MyException("03","业务错误"),
    /**
     * 提示级错误. ErrorCode : 04
     */
    InfoException("04", "提示级错误"),
    /**
     * 数据库操作异常. ErrorCode : 020001
     */
    DBException("020001","数据库操作异常"),
    /**
     * 参数验证错误. ErrorCode : 040001
     */
    ParamException("040001","参数验证错误");

    private String code;

    private String msg;

    ResultEnum(String code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public String getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}

```