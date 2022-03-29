---
title: "Common Cross Site Scripting(XSS) and ways to prevent it"
date: 2021-09-15T10:57:25+08:00
description: "introduction of Cross Site Scripting(XSS) and how to prevent it"
menu:
  sidebar:
    name: Cross-Site Scripting(XSS)
    identifier: Cross-Site-Scripting
    parent: Web-Development
    weight: 10
<!--- hero: images/xss3.jpg --->
---
Cross-site Scripting (XSS) is a client-side code injection attack. It's one of the most commonly seen website attacks, even Facebook and Google has been exposed having XSS vulnerabilities. 

If successful, XSS vulnerabilities can be exploited to manipulate or steal cookies, create requests that can be mistaken for those of a valid user, compromise confidential information, or execute malicious code on end-user systems. 

As a web developer, it's better to know how it works and avoid using those codes than to revise the project after the vulnerabilities being detected or targeted. There are many kinds of XSS attacks often injecting through HTML, JavaScript, CSS, XML and URL. Here are some basic attacks and the solutions to fix them.

## Basic XSS attack :space_invader:
the malicious payload can be embedded in a URL (e.g. in query strings of GET requests). An attacker can trick a victim, via a phishing attack, to click on a link with vulnerable input which has been altered to include attack code and sent to the legitimate server. The injected code is then reflected to the user's browser which executes it.
A malicious payload might look like this in input that requires for the URL:
```java
http://example.com/test=<script>alert('xss payload');</script>
```
another type of attacks doesn't use script tags at all, but instead uses an embedded tag with a javascript attribute:
```java
http://example.com/test= <img src=javascript:alert('xss payload')/>
```
Converting tags to URL encode could be a tricky one, cause it's not that obvious to find out, it looks just like a normal URL:  
```java
%3Cscript%3Ealert(%27You%20have%20been%20hacked%27)%3B%3C%2Fscript%3E
```
if decoding that URL you'll see it actually represents `<script>alert('You have been hacked');</script>` (you can easily encode/decode any characters in website like [Encoder](http://meyerweb.com/eric/tools/dencoder/))

The attack happens if the browser or the application decodes that URL and renders it, whereas if the page treats the payload in the same URL syntax, there won't be an XSS.

## 2. How to prevent XSS :ok_woman:
When developing web application, it's important to design the system to never assume user input is valid or "clean". Ensure proper filtration and encode user-supplied data is recommended.

### 1.Validating all input
When validating a form, verify that all inputs match the strictest definition of valid value, do this at both frontend and backend programs for better protection, because an attacker can easily "disabled javascript" on the browser, so the frontend validation won't be functioning. 

#### Example of using JQuery Validation API
```javascript
<script type="text/javascript">
    $(document).ready(function() {
        $("#register-form").validate({
            rules : {
                "userName" : {
                    required : true,
                    englishNumberValidate : true
                },
            invalidHandler:function(e,validator){
              for (var i=0;i<validator.errorList.length;i++){
                    var id = validator.errorList[i].element.id;
                    $('#'+id).val(""); //clear out inappropriate value
              }
                }
     }
    })
});

$.validator.addMethod("englishNumberValidate",
 englishNumberValidate,"Please insert English or Numbers only");

function englishNameValidate(value, element) {
  var result = this.optional(element);
  var reg = new RegExp("^[a-zA-Z]+([ ][a-zA-Z]+)*$");

  if (!result && reg.test(value)) {
    jQuery(element).val(value.toUpperCase());
    result = true;
  }
  return result;
}
</script>
```
For the frontend, [jQuery Validation Plugin](https://github.com/jquery-validation/jquery-validation) creates a custom validation for your inputs. Make sure after validating, inappropriate values should not remain in the input, clear them out or the browser might render it while refreshing the page, doing this could prevent malicious payload that is URL encoded.

#### Example of using Hibernate Validator
```java
/** Register Entity */
    @Id
    @Column(name = "UserId", unique = true, nullable = false, length = 50)
    @Length(max=50,message = "Maximum length of Id is 50 characters")
    @Pattern(regexp="^[\\w.]+$",message = "Only English, numbers and "_" "." is allowed")
    public String getUserId() {
        return this.userId;
    }

```
For backend validation, [this API](https://docs.jboss.org/hibernate/stable/validator/api/) is useful as it contains constraint annotations like `@NotNull`, `@Size` and `@Min`,`@Max`, just use them for setting the rules to pass the validation of each input.
```java
/** Register Controller */
    @PostMapping("")
    public String register(@Validated @ModelAttribute UserDef userdef, BindingResult bindingResult, HttpSession session,
            RedirectAttributes redirectAttributes) {
        boolean isCaptchaValid = captchaService.isValidate(session,true);
        if ( !isCaptchaValid || bindingResult.hasErrors()) {
            StringBuilder errSb = new StringBuilder("Error Columns：");
            if(bindingResult.hasErrors()){
                bindingResult.getFieldErrors().forEach(f -> {
                    errSb.append(f.getDefaultMessage()).append("\\n");
                    try {
                        //clear out inappropriate value
                        BeanUtils.setProperty(userdef, f.getField(), null);
                    } catch (Exception e) {
                        LOGGER.error("Reset field({}) value fail.", f.getField());
                    }
                });
            }
            redirectAttributes.addFlashAttribute("user",userdef);
            redirectAttributes.addFlashAttribute("regErrMsg",errSb.toString());
            return "redirect:/register";
        }
```
In the controller class, Use `BindingResult.hasErrors()` to validate if the input(column) matches what you've set in the entity, use `getDefaultMessage()` to get the custom message and save it as a String, then return it using `redirectAttributes` to writes it to the page.

### 2.Encode all input 
Character include `<`,`>`,`&`,`"`,`'` should be encoded to special hexadecimal digits to prevent switching into any execution context, such as script and style. [HTML Encode](https://emn178.github.io/online-tools/html_encode.html) is designed for this purpose, for example, it converts `<` to &lt, when rendering on client-side, &lt will be automatically transformed to `<`.

There are many functions or APIs to do HTML Encode, like `escapeHtml()`and`htmlspecialchars()` for javaScript; As for backend, Java has [org.apache.commons.text.StringEscapeUtils](https://commons.apache.org/proper/commons-text/javadocs/api-release/org/apache/commons/text/StringEscapeUtils.html) library which could escapes and unescapes special characters for Java, JavaScript, HTML and XML.

