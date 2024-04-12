### Data journey

#### Legend

`Client` is the end client talking to Bürokratt.

`Widget` is a chat widget developed by Bürokratt, but installed by institutions themselves.

`Ruuter` is one of Bükstack's core components used by Bürokratt.

`Resql` is one of Bükstack's core components used by Bürokratt.

`SSE` is one of Bükstack's core components used by Bürokratt.

`Rasa` is an external software to provide basic chat services and create rules for more complicated chat flows.

`RelationalDB` is a Postgres database by default, but could be potentially switched to any other relational databases.

`NoSQL` is OpenSearch by default, but could be potentially switched to other NoSQL databases.

`External` is considered as a service provided by any third-party service providers. They can be LLMs, classifiers, etc.

#### Use case nr 1 - End Client gets their response from Rasa

This is an overview of data journey when the End Client chats with Bürokratt and gets their response from local Rasa.

External service providers are missing from this sequence diagram as they do not have any role in such use cases.

```mermaid
sequenceDiagram
  participant Client
  participant Widget
  participant Ruuter
  participant Rasa
  participant Resql
  participant RelationalDB
  participant NoSQL
  participant SSE

  autonumber
  Client->>+Widget: Hello
  Widget->>+Ruuter: Hello
  Ruuter->>+Rasa: Hello
  Rasa-->>-Ruuter: Hello. How can I help you?
  Ruuter->>RelationalDB: How can I help you?

  loop Continuously fetch for new messages
    loop Check for new message notifications
      note over SSE,NoSQL: Cheap continuous calls
      SSE->>+NoSQL: New messages?
      NoSQL-->>-SSE: Boolean true/false
      note over Widget,SSE: Notify Widget only if there are any new messages to be fetched. This is to prevent exhausting Ruuter and other backend systems with unnecessary processes.
      SSE->>Widget: *There are new messages for Chat ID XYZ*
    end

    Widget->>Ruuter: *Send new messages for Chat ID XYZ*
    Ruuter->>+Resql: *Send new messages for Chat ID XYZ*
    Resql->>+RelationalDB: *Send new messages for Chat ID XYZ*
    RelationalDB-->>-Resql: How can I help you?
    Resql-->>-Ruuter: How can I help you?
    Ruuter-->>-Widget: How can I help you?
  end
  
  Widget-->>-Client: The book Kevade was written by an Estonian writer Oskar Luts
```

#### Use case nr 2 - End Client gets their response from and External source

This is an overview of data journey when the End Client chats with Bürokratt and gets their response from an External service providers due to local Rasa not being able to respond.


```mermaid
sequenceDiagram
  participant Client
  participant Widget
  participant Ruuter
  participant Rasa
  participant Resql
  participant RelationalDB
  participant NoSQL
  participant SSE
  participant External

  autonumber
  Client->>+Widget: Who wrote the book "Kevade"?
  Widget->>+Ruuter: Who wrote the book "Kevade"?
  Ruuter->>+Rasa: Who wrote the book "Kevade"?
  Rasa-->>-Ruuter: *No response*

  note over Ruuter,External: All sensitive information, including Chat ID, JWT tokens, etc. are removed by Ruuter when making requests to External service providers

  Ruuter->>+External: Who wrote the book "Kevade"?
  External-->>-Ruuter: The book Kevade was written by an Estonian writer Oskar Luts

  note over Ruuter,RelationalDB: Ruuter now again uses sensitive information in next steps due to making them again only in secure, trusted environment

  Ruuter->>RelationalDB: The book Kevade was written by an Estonian writer Oskar Luts

  loop Continuously fetch for new messages
    loop Check for new message notifications
      note over SSE,NoSQL: Cheap continuous calls
      SSE->>+NoSQL: New messages?
      NoSQL-->>-SSE: Boolean true/false
      note over Widget,SSE: Notify Widget only if there are any new messages to be fetched. This is to prevent exhausting Ruuter and other backend systems with unnecessary processes.
      SSE->>Widget: *There are new messages for Chat ID XYZ*
    end

    Widget->>Ruuter: *Send new messages for Chat ID XYZ*
    Ruuter->>+Resql: *Send new messages for Chat ID XYZ*
    Resql->>+RelationalDB: *Send new messages for Chat ID XYZ*
    RelationalDB-->>-Resql: The book Kevade was written by an Estonian writer Oskar Luts
    Resql-->>-Ruuter: The book Kevade was written by an Estonian writer Oskar Luts
    Ruuter-->>-Widget: The book Kevade was written by an Estonian writer Oskar Luts
  end
  
  Widget-->>-Client: The book Kevade was written by an Estonian writer Oskar Luts
```
