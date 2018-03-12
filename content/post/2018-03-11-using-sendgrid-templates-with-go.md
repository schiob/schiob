---
title: Using Sendgrid templates with go
subtitle: A simple implementation to send mails
date: 2018-03-11T21:52:22.006Z
tags:
  - go
  - mail
---
Last week I have to write an implementation to send emails with go, so here is a simple example of what I end up doing using the Sendgrid templates.

Use the official go library [sendgrid-go](https://github.com/sendgrid/sendgrid-go), it is really simple to use and have a great set of examples [here](https://github.com/sendgrid/sendgrid-go/blob/master/USE_CASES.md).

### Simple Email
First install the package.

`go get github.com/sendgrid/sendgrid-go`

Import it and use the sendrid Helper Class **mail** to send a simple message:

```go
package main

import (
  "fmt"

  "github.com/sendgrid/sendgrid-go"
  "github.com/sendgrid/sendgrid-go/helpers/mail"
)

func main() {
  from := mail.NewEmail("Example User", "test@example.com")
  to := mail.NewEmail("Example User", "test@example.com")
  subject := "Sending with SendGrid is Fun"
  plainTextContent := "and easy to do anywhere, even with Go"
  htmlContent := "<strong>and easy to do anywhere, even with Go</strong>"
  message := mail.NewSingleEmail(from, subject, to, plainTextContent, htmlContent)
  client := sendgrid.NewSendClient("YOUR_SENDGRID_API_KEY")
  response, _ := client.Send(message)
  fmt.Println(response.StatusCode)
  fmt.Println(response.Body)
  fmt.Println(response.Headers)
}
```

The first lines of the main function prepare the from, to and subject of the mail. And the next 2 are the content in plain text and html.

Then you create the mail using the **NewSingleEmail** function, prepare your client with the **Sendgrid api key** of your account and send the mail.

You can use the response object for check the status, headers or body of the API response. And please take proper care of the error that I ignore for simplicity of the code.

### Using Templates
In order to use the templates first you have to create one in your [Transactional Templates](https://sendgrid.com/templates) section in your account. More information in the sendgrid [web page](https://sendgrid.com/solutions/transactional-email-templates/).

One you create the template we have to replace some lines of the last code.

```go
package main

import (
  "fmt"

  "github.com/sendgrid/sendgrid-go"
  "github.com/sendgrid/sendgrid-go/helpers/mail"
)

func main() {
  from := mail.NewEmail("Example User", "test@example.com")
  to := mail.NewEmail("Example User", "test@example.com")
  subject := "Sending with SendGrid is Fun"
  content := mail.NewContent("text/html", "I'm replacing the <strong>body tag</strong>")
  m := mail.NewV3MailInit(from, subject, to, content)
  m.Personalizations[0].SetSubstitution("-name-", "Example User")
  m.Personalizations[0].SetSubstitution("-city-", "Denver")
  m.SetTemplateID("13b8f94f-bcae-4ec6-b752-70d6cb59f932")
  client := sendgrid.NewSendClient("YOUR_SENDGRID_API_KEY")
  response, _ := client.Send(m)
  fmt.Println(response.StatusCode)
  fmt.Println(response.Body)
  fmt.Println(response.Headers)
}
```

The html in the **content** variable will replace the <%body%> tag in the template. And the **-name-** and **-city-** are also substitution tags.

You can add more personalizations to the mail like an array of tos but you can check those kind of usecases in the package github [examples](https://github.com/sendgrid/sendgrid-go/blob/master/USE_CASES.md#personalizations).
