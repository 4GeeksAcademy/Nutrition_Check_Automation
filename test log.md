# Test Log

## 

\---

Test ID        Type           Scenario /      Expected result              Status
input

\---

TC-FUN-001     Functional     Known valid      Plain-text verdict returned; Pass
                              barcode with     healthy only if both  
                              complete         readings are healthy;  
                              nutrition data   formatted sections present  
                              and Nutri-Score  
                              A/B



TC-FUN-002     Functional     Known product    Verdict is the worse of      Pass
                              with one high    moderate traffic-light level
                              nutrient and     and moderate grade; concern  
                              Nutri-Score C    score is 1

                           



TC-FUN-003     Functional     Product with     Traffic-light level          Pass
                              2--3 high        unhealthy; final verdict  
                              nutrients        cannot be better than  
                                               unhealthy



TC-INT-001     Integration    Valid barcode    HTTP 200 and API             Pass
                              sent to live     `status: 1`; product fields  
                              Open Food Facts  parsed correctly  
                              API



TC-INT-002     Integration    Unknown numeric  API not-found response is    Pass
                              barcode sent to  captured and routed;  
                              live API         workflow does not crash



TC-ERR-001     Error          Unknown numeric  Plain-text product-not-found Pass
                              barcode          message; no AI call



TC-ERR-002     Error          Malformed        Plain-text invalid-barcode   Pass
                              barcode such     API is not called
                              as message;   
                              `ABC123`



TC-ERR-003     Error          Empty request    Plain-text                   Pass
                              body `{}`        usage/invalid-barcode  
                              message; 
                              workflow does not  
                              crash



TC-ERR-004     Error          Existing         Insufficient-data message;   Pass
                              product with     no  
                              missing/empty    healthy/moderate/unhealthy  
                              required         classification  
                              nutrition  
                              fields



TC-ERR-005     Error          Nutri-Score      Verdict falls back to        Pass
                              missing or       traffic-light count alone  
                              unknown but all  
                              required  
                              nutrient values  
                              present



TC-PER-001     Performance    Run 10          Record average, minimum,     Pass
                              sequential      maximum and failures;  
                              valid-barcode   establish a baseline (no  
                              requests and    performance claim until  
                              record          measured)  
                              end-to-end  
                              durations





