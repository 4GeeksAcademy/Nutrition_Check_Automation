# Test Log

---

| Test ID    | Type        | Scenario / Input                                                         | Expected result                                                                                           | Status |
| ---------- | ----------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- | ------ |
| TC-FUN-001 | Functional  | Known valid barcode with complete nutrition data and Nutri-Score A/B     | Plain-text verdict returned; healthy only if both readings are healthy; formatted sections present        | Pass   |
| TC-FUN-002 | Functional  | Known product with one high nutrient and Nutri-Score C                   | Verdict is the worse of moderate traffic-light level and moderate grade; concern score is 1               | Pass   |
| TC-FUN-003 | Functional  | Product with 2–3 high nutrients                                          | Traffic-light level unhealthy; final verdict cannot be better than unhealthy                              | Pass   |
| TC-INT-001 | Integration | Valid barcode sent to live Open Food Facts API                           | HTTP 200 and API `status: 1`; product fields parsed correctly                                             | Pass   |
| TC-INT-002 | Integration | Unknown numeric barcode sent to live API                                 | API not-found response is captured and routed; workflow does not crash                                    | Pass   |
| TC-ERR-001 | Error       | Unknown numeric barcode                                                  | Plain-text product-not-found message; no AI call                                                          | Pass   |
| TC-ERR-002 | Error       | Malformed barcode such as message; `ABC123`                              | Plain-text invalid-barcode API is not called                                                              | Pass   |
| TC-ERR-003 | Error       | Empty request body `{}`                                                  | Plain-text usage/invalid-barcode message; workflow does not crash                                         | Pass   |
| TC-ERR-004 | Error       | Existing product with missing/empty required nutrition fields            | Insufficient-data message; no healthy/moderate/unhealthy classification                                   | Pass   |
| TC-ERR-005 | Error       | Nutri-Score missing or unknown but all required nutrient values present  | Verdict falls back to traffic-light count alone                                                           | Pass   |
| TC-PER-001 | Performance | Run 10 sequential valid-barcode requests and record end-to-end durations | Record average, minimum, maximum and failures; establish a baseline (no performance claim until measured) | Pass   |
