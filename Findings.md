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
  2. Replace the value with an undefined value.
  3. Sent the modified request to the API endpoint.
  4. Observe a successful response with no errors.
  ___
  ### Expected Behavior
  The server should validate all incoming 'event_type' values against a strict allowlist. Any unexpected or unrecognized values
  should trigger a '4XX' error response.
  ### Actual Behavior
  The server accepted the modified 'event_type' and processed the event without performing validation or returning an error.
  ___
  ### Recommendation
  Implement strict server-side validation for all 'event_type' values using a predefined list of expected names.
  Reject or log any requests with unrecognized types.
  ___  
  ## 4. Lack of Validation on Core Metadata Fields
  **Severity:** Medium  
  **Status:** Patched  
  ___
  ### Summary
  During testing, I modified key metadata fields in the event submission payload, including 'country_code', 'platform' , and 'environment'. I changed 'country_code' to an invalid value, 'platform' to 'Android' instead of 'iOS', and 'environment' to a fake value. Despite these changes, the server responded with a '200 OK' and accepted the events without any error or validation failure.
  ___
  ### Technical Details
  The server failed to validate core metadata fields in the incoming event payload. These fields, which may be used for analytics, feature access control, or environment-specific logic, were accepted even when set to invalid or non-standard values. This lack of validation could lead to inconsistent backend behavior, spoofed environments, or corrupted data.
  ___
  ### Steps to Reproduce
  1. Intercepted a valid event submission.
  2. Modified the 'country_code' field to  'ZZ'
  3. Modified the 'platform' field to 'Android' (when iOS was expected)
  4. Modified the 'environment' field to 'Staging123'
  5. Sent request to the server
  6. Observed a successful '200 OK' response with no errors
  ___
  ### Expected Behavior
  The server should validate incoming metadata fields like 'country_code', 'platform', and 'environment' against a list of expected values. Requests containing invalid or unexpected entries should be rejected or logged.
  ### Actual Behavior
  The server accepted all modified metadata fields without validation or error, processing them as if they were legitimate values.
  ___
  ### Recommendation
  Implement strict server-side validation for metadata fields, reject or log any requests with unexpected inputs.
  ___  
  ## 5. Lack of Validation on User ID Field  
  **Severity:** High  
  **Status:** Patched  
  ___
  ## Summary
  During testing, I modified the 'user_id' field within the event payload by adding a character to an otherwise valid ID.
  Despite the modification, the server responded with a '200 OK' and accepted the event without any errors.
  ___
  ### Technical Details
  The API endpoint failed to validate the authenticity of the 'user_id' field. This allowed modified, tampered or unregistered user IDs to be accepted without verification. This behavior can lead to user spoofing, fake activity generation, or manipulation of analytics and account-related data.
  ### Steps to Reproduce
  1. Intercepted a valid event request
  2. Modified the 'user_id' field by changing a character to the original ID
  3. Forwarded the modified request to the API endpoint
  4. Observed a succesful '200 OK' response with no errors.
  ___
  ### Expected Behavior
  The server should validate that the 'user_id' matches a real, authenticated session associated with the current user or device.
  Amy tampered, unknown, or unregistered 'user_id' should be rejected.
  ### Actual Behavior
  The server accepted the modified 'user_id' and accepted the request without performing any validation or errors.
  ___
  ### Recommendation
  Implement strict server-side validation of the 'user_id' field. The server should ensure that the 'user_id' matches
  the authenticated user and device and events originating from unknown or tampered user IDs are rejected and logged.
  ___
  ## 6. Critical Lack Validation on Environment and User ID Fields
  **Severity:** High  
  **Status:** Patched  
  ___
  ### Summary
  During testing, I modified both the 'environment' and 'user_id' fields inside a valid POST request to an endpoint.
  I set 'environment' to 'admin' and slightly altered the 'user_id' value. Despite these changes, the server responded with '200 OK'
  and accepted the event without any error or verification.
  ___
  ### Technical Details
  The backend failed to enforce validation on two critical fields, 'environment' and 'user_id'. This allowed forged values to be
  accepted without authentification checks. Events were successfully published under invalid or spoofed user and environmentat contexts, introducing trust and integrity issues.
  ___
  ### Steps to Reproduce
  1. Intercepted a valid POST request to an endpoint
  2. Modified the 'environment' field to 'admin'
  3. Slightly altered the 'user_id' field to tampered version
  4. Sent the request to the server
  5. Observed a '200 OK' response with the event processed successfully
  ___
  ### Expected Behavior
  The backend should validate sensitive fields like 'environment' and 'user_id', should only accepted predefined values, and the
  'user_id' should match a valid registed user tied to the current session. Any manipulation should result in rejection or logging.
  ### Actual Behavior
  The server accepted forged environment and user identifiers without validation. Events were successfully modified under admin-like
  contexts and invalid user IDs.
  ### Recommendation
  Implement strict server-side validation to only allow predefined values for fields like 'environment', cross check 'user_id'
  values with active, authenticated sessions, and reject requests with any invalid, unknown or unauthorized field values.
  





  
     





  
  




  



  
  
  


 





  


  


  





  


  
  
  
  

  
   
  
  
