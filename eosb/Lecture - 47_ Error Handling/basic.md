## if we want to implement custom error controller or error handler  without use BasicErrorController
```
1.implements ErrorController 
2.override errorHtml(),and error() mehtod method name can be change ;
What is ErrorAttributes?
```
## ErrorAttributes is a Spring Boot interface that provides error-related data when an error happens.

👉 It supplies information like:
```
HTTP status

Error message

Exception name

Request path

Timestamp

Spring Boot internally uses this to build:

Error JSON (REST)

Error Model (HTML pages)

🔹 Default Error Attributes (Provided by Spring Boot)

When an error occurs, Spring Boot creates a map like this:

{
  "timestamp": "2025-12-14T10:30:15.123+06:00",
  "status": 404,
  "error": "Not Found",
  "message": "No handler found",
  "path": "/api/test"
}


This data comes from DefaultErrorAttributes.
```
## All type of error handle this customErrorHandler:
```
@Controller
@RequestMapping("/error")
public class CustomErrorController implements ErrorController {

    @Autowired
     private ErrorAttributes errorAttributes;
// if want to response html view is method is called 
// @ResponseBody
 @GetMapping(produces = MediaType.TEXT_HTML_VALUE)
    public String errorHtmlpage(HttpServletRequest request, Model model)
    {

        WebRequest webRequest = new ServletWebRequest(request);

     Map<String,Object> map = errorAttributes.getErrorAttributes(webRequest,
             ErrorAttributeOptions.of(
            MESSAGE,STACK_TRACE,PATH,STATUS,ERROR
     ));
//     model.addAttribute(map);
        map.forEach(model::addAttribute);
return "error2";
//        return """
//        <h1>Something went wrong</h1>
//        <p><b>Timestamp:</b> %s</p>
//        <p><b>Status:</b> %s</p>
//        <p><b>Error:</b> %s</p>
//        <p><b>Message:</b> %s</p>
//        <p><b>Path:</b> %s</p>
//        <pre><b>Stack Trace:</b>\n%s</pre>
//        """
//                .formatted(
//                        map.get("timestamp"),
//                        map.get("status"),
//                        map.get("error"),
//                        map.get("message"),
//                        map.get("path"),
//                        map.get("trace")
//                );
    };


// if want to response body or json   is method is called 
    @ResponseBody
    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> restErrorHandler(HttpServletRequest request)
    {
        WebRequest webRequest = new ServletWebRequest(request);

        Map<String,Object> map = errorAttributes.getErrorAttributes(webRequest, ErrorAttributeOptions.of(
                MESSAGE,STACK_TRACE,PATH,STATUS,ERROR
        ));
        map.put("extra-infa","this is extra-info");
        map.put("info","this is info");

        return ResponseEntity.status((Integer) map.get("status")).body(map);

    }

}
```
