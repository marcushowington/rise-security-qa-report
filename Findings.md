# Findings Report - Rise App (v4 Pre-Launch) 

## Test Environment  
- **Platforms Tested:** iOS Public App Store & TestFlight versions)
- **Devices Used:** iPhone (used as proxy) and Macbook (interception and analysis)
- **Network Configuration:** iPhone traffic proxied through Burp Suite using Macbook setup
- **Testing Timeline:** April 2025 - June 2025
- **Scope:** API Behavior, authentification, and endpoint exposure

**Note:** Internal version numbers, authentification tokens, and any sensitive infrastructure references have been redacted or generalized to respect the confidentiality of Rise.  

## Tools Used  
- **Burpe Suite Community Edition** - Intercepting HTTPS traffic
- **Postman** - API Exploration & Testing
- **macOS Terminal Tools** For DNS/IP lookups
- **iPhone Proxy via Macbook** - Traffic routing and analysis
  ___
  ## Key Findings
  ## 1. Tampered Device ID Accepted  
  **Severity:** Medium  
  **Status:** Patched  
  ___
  ### Summary  
  A single character was modified in a valid 'device_id' and sent to the '**/api-endpoint**'. Despite this tampering, the server returned a successful 'HTTP/2 200 OK' response without rejecting the request, indicating a lack of validation.
  ___
  ### Technical Details
  The API should validate device identifiers to ensure they match registered devices. In this case, changing a single character still resulted in a valid response.
  
  **Steps to Reproduce:**  
  1. Intercepted traffic using Burp Suite
  2. Modified a known 'device_id' (example: '123457')
  3. Sent modified request to the '**/api-endpoint**'
  4. Observed a successful response with no errors
  ___
  ### Expected Behavior
  The server should reject tampered 'device_id' values and return a '4XX' status code if validation fails.
  ### Actual Behavior
  The tampered identifier was accepted without validation, potentially exposing the app to unauthorized data pollution or spoofing risks.
  ___
  ### Recommendation
  Enforce strict server-side validation for all incoming device identifiers using a known-safe registry or token-based authentication system.
  ___
  ## 2. Missing Device ID Accepted
  **Severity:** Medium  
  **Status:** Patched
  ___
  ### Summary
  During API testing of a redacted endpoint, I removed the 'device_id' field from a standard request payload. The server accepted the incomplete request, returning '200 OK' instead of rejecting the input.
  ___
  ### Technical Details
  The API endpoint did not enforce validation for required fields. The 'device_id' parameter is designed to uniquely identify devices but was not treated as mandatory during request.
  ___
  ### Steps to Reproduce
  1. Intercepted traffic using Burp Suite
  2. Removed the 'device_id' parameter from the JSON body.
  3. Sent modified request to the API endpoint.
  4. Observe a successful response with no errors
  ___
  ### Expected Behavior
  The server should reject requests with missing required identifiers with a '4XX' status code if validation fails.
  ### Actual Behavior
  Requests missing the 'device_id' were accepted without validation.
  ___
  ### Recommendation
  Implement server-side validation that enforces the presence of the 'device_id' field for all event submissions. Requests missing this field should be rejected with an appropriate error response.  
  ___
  ## 3. Arbitrary Event Type Accepted
  **Severity:** Medium  
  **Status:** Patched
  ___
  ### Summary
  During testing, I changed the 'event_type' field in a request to an arbitrary string ('bug_test') The server accepted and processed this unrecognized without validation, returning a '200 OK' response.
  ___
  ### Technical Details
  The endpoint lacked proper validation of the 'event_type' parameter. This could allow injection of unexpected event categories, corrupt analytics data, or trigger undefined backend behavior depending on how event types are handled.
  ___
  ###  Steps to Reproduce
  1. Intercepted a valid 'event_type' request.
  2. Replace the value with an undefined string.
  3. Sent the request
  4. Observe a successful response with no errors.
___  

  
   
  
  
