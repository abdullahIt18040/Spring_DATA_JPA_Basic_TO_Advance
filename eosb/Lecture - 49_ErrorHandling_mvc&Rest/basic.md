## ExceptionHandler annotation base Rest and MVC Seperate

```
Handld all MVC all exception
---------------------------------------------------------------
@ControllerAdvice(annotations = MVCControllerIdentifier.class)
public class GloblMvcExceptionHandler {


    @ExceptionHandler(ResourceNotFountException.class)
    public String resourceNotFound(ResourceNotFountException ex, Model model)
    {
        model.addAttribute("error",ResourceNotFountException.class.getSimpleName());
        model.addAttribute("message",ex.getMessage());

        return "resource_not_found";

    }
}

--------------------------------------------------------------------------------
@RestControllerAdvice(annotations = RestControllerIdentifier.class)
public class GlobalExceptionHandler {

    @Autowired
   private ErrorAttributes errorAttributes;

   @ResponseBody
   @ResponseStatus(HttpStatus.NOT_FOUND)
    @ExceptionHandler(ResourceNotFountException.class)
    public ErrorResponse resourceNotFound(ResourceNotFountException ex)
    {
        return new ErrorResponse(HttpStatus.NOT_FOUND.value(),
                HttpStatus.NOT_FOUND.toString(),
                ex.getMessage()
        );
    }








    @ResponseBody
    @ExceptionHandler(MathExeption.class)
    public ResponseEntity<?>mathExceptionHandler(Exception ex)
    {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(
                Map.of("timestamp", Instant.now(),
                        "status",HttpStatus.INTERNAL_SERVER_ERROR,
                        "message",ex.getMessage(),
                        "error",MathExeption.class.getSimpleName()

                ) );



    }



    @ResponseBody
    @ExceptionHandler(AccessDenide.class)
    public ResponseEntity<?>accessDenideExceptionHandler(HttpServletRequest request, Exception ex)
    {
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(
                Map.of("timestamp", Instant.now(),
                        "status",HttpStatus.FORBIDDEN,
                        "message",ex.getMessage(),
                        "error",AccessDenide.class.getSimpleName(),
                        "path","while try to access url[%s]".formatted(request.getRequestURI())

                ) );



    }



    @ResponseBody
    @ExceptionHandler(PageNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String pageNotFoundExceptionHandler(
            PageNotFoundException ex,
            Model model) {

        model.addAttribute("error", ex.getMessage());
        return "page_not_found_error";
    }



    @ResponseBody
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler(Throwable.class)
    public String errorHandler(Throwable ex)
    {

//
//        WebRequest webRequest = new ServletWebRequest(request);
//
//        Map<String,Object> map = errorAttributes.getErrorAttributes(webRequest,
//                ErrorAttributeOptions.of(
//                        MESSAGE,STACK_TRACE,PATH,STATUS,ERROR
//                ));
//   model.addAttribute(map);
//        map.forEach(model::addAttribute);
//        model.addAttribute("error",ex.getMessage());
        return "exception is :"+ex.getMessage();
    }
}
```
## ErrorResponseEntity

```
@Setter
@Getter
public class ErrorResponse {

    private Date timestamp = new Date();
    private String message;
    private String status;
    private int statuscode;

    public ErrorResponse(int statuscode, String status, String message) {
        this.timestamp = new Date();
        this.statuscode = statuscode;
        this.status = status;
        this.message = message;
    }

}
```
## CustomAnnotation

 For RestController
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface RestControllerIdentifier {
}

 For MVC Controller
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MVCControllerIdentifier {
}
```

## controller 
For RestController

```
-------------------------------------------------------------
@RestController
@RestControllerIdentifier
@RequestMapping("/api/v1")
public class StudentController {
    public StudentController()
    {
      students.add( new Student(1,"AND",3.52f)) ;
        students.add( new Student(2,"AND1",3.52f)) ;
        students.add( new Student(3,"AND3",3.522f)) ;


    }

private List<Student>students = new ArrayList<>();
@GetMapping("/students")
 public List<Student>students()
 {

     return students;
 }
 @GetMapping("/student")
 public Student getStudent(@RequestParam("id") int id)
 {
   return students.stream().filter(
            s->s.id()==id).findFirst().orElseThrow(()-> new ResourceNotFountException("" +
           "student not found id id [%d]\".formatted(id) "));

}

}


----------------------------------------------------
For mvc controller
@Controller
@MVCControllerIdentifier
public class StudentMVCController {
    public StudentMVCController()
    {
        students.add( new Student(1,"AND",3.52f)) ;
        students.add( new Student(2,"AND1",3.52f)) ;
        students.add( new Student(3,"AND3",3.522f)) ;


    }

    private final List<Student> students = new ArrayList<>();
    @GetMapping("/students")
    public List<Student>students()
    {

        return students;
    }
    @GetMapping("/student")
    public String getStudent(@RequestParam("id") int id, Model model)
    {
        var student= students.stream().filter(
                s->s.id()==id).findFirst().orElseThrow(()-> new ResourceNotFountException("" +
                "student not found id id [%d].formatted(id) "));
        model.addAttribute("student",student);
        return "student";


    }
}
```





