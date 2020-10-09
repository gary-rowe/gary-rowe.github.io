---
author: Gary Rowe
title: How to include Markdown with Freemarker in Dropwizard
layout: post
tags:
  - Dropwizard
  - Freemarker
  - HowTo
  - Markdown
redirect_from: /agilestack/2012/11/05/how-to-include-markdown-with-freemarker-in-dropwizard/
---

One common problem that web developers face is the elegant handling of rich text received from their users which should be rendered as HTML. Obviously, accepting raw HTML from unknown sources is a major security risk. Fortunately, this problem has been solved through the use of [Markdown][2] which requires only an easily readable text document from your users. If you’re looking for a portable rich text solution for documentation and requirements specifications then Markdown is a great solution.

In order to include Markdown into your own projects you need only use a suitable Markdown parser, and an easy to use one is [pegdown][3]. This is available in Maven central under the following co-ordinates:

```xml
<dependency>
  <groupId>org.pegdown</groupId>
  <artifactId>pegdown</artifactId>
  <version>1.1.0</version>
</dependency>
```

In order to use it in Dropwizard just add it to the model that backs your View like this:

```java
public String getHtml() throws IOException {

  // Hard coded but could come from anywhere
  String markdown = "## Example Markdown\n" +
    "\n" +
    "This is a paragraph\n" +
    "\n" +
    "This is another\n" +
    "\n" +
    "[This is a link](http://example.org)";

  // New processor each time due to pegdown not being thread-safe internally
  PegDownProcessor processor = new PegDownProcessor();

  // Return the rendered HTML
  return processor.markdownToHtml(markdown);

}
```

Then you simply include the Markdown output into your FreeMarker template like so:

```html
<#-- @ftlvariable name="" type="org.example.MyView" -->
<!DOCTYPE html>
<html lang="en">
<head>
</head>
<body>

${html}

</body>
</html>
```

Hope this helps someone!

 [1]: https://twitter.com/share
 [2]: http://daringfireball.net/projects/markdown/
 [3]: https://github.com/sirthias/pegdown