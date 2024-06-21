### Data journey

This is to give an overview of a data journey when the End Client is trying to use common services provided by Bürokratt.

`Service` in this case is considered as a standard Ruuter DSL. It's separated on a sequance diagram as it could technically also be any external REST service.

#### User case nr 1 - company's name can be exactly detected based on End Client's initial text input

```mermaid
sequenceDiagram

participant Client
participant Widget
participant Ruuter
participant Rasa
participant OpenSearch
participant Service
participant DataMapper

autonumber
Client ->>+ Widget: "Who are the board members of Company ABC123?"
    Widget ->>+ Ruuter: "Who are the board members of Company ABC123?"
        Ruuter ->>+ Rasa: "Who are the board members of Company ABC123?"
        Rasa -->>- Ruuter: ":service_company_board_members"

        loop Find matching endpoint for intent
            Ruuter -> Ruuter: ":service_company_board_members"
            Ruuter --> Ruuter: "/company/board/members"
        end

        Ruuter ->>+ Service: "Who are the board members of Company ABC123?"

        loop Multiple service-specific steps to get the result
            loop Get company name from user's input
                Service ->>+ OpenSearch: /company/name "Who are the board members of Company ABC123?"
                OpenSearch ->>- Service: "Company ABC123"
            end

            loop Get company's board members
                Service ->>+ OpenSearch: /company/board/members "Company ABC123"
                OpenSearch ->>- Service: ["Mr. X", "Mr. Y"]
            end

            loop Create a meaningful response
                Service ->>+ DataMapper: /service/company/board/members "["Mr. X", "Mr. Y"]"
                DataMapper ->>- Service: "The board members of Company ABC123 are Mr. X and Mrs. Y"
            end
        end

        Service -->>- Ruuter: "The board members of Company ABC123 are Mr. X and Mrs. Y"
    Ruuter -->>- Widget: "The board members of Company ABC123 are Mr. X and Mrs. Y"
Widget -->>- Client: "The board members of Company ABC123 are Mr. X and Mrs. Y"
```
