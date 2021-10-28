# jumpcloud-interview
Password hash assignment Test plan <br/>
Acceptance Criteria <br/>

- When launched, the application should wait for http connections. **Verified**.<br/><br/>

Tested: Confirmed that an error occurs (‘port must be set’) if you attempt to run the application without HTTP connections (without first opening the port).<br/>

- It should answer on the PORT specified in the PORT environment variable. **Verified**. <br/>

Tested: Opened port 8080, ran stats command (curl http://127.0.0.1:8088/stats). **Verified** received an error that the connection was refused. <br/>

Tested: Opened port 8088, ran stats command (curl http://127.0.0.1:8088/stats). **Verified** stats returned. <br/>

 - A POST to /hash should accept a password. It should return a job identifier immediately. It should then wait 5 seconds and compute the password hash. The hashing algorithm should be SHA512. **Partial failure.** <br/>

Tested: ran command curl -X POST -H "application/json" -d '{"password":"angrymonkey"}' http://127.0.0.1:8088/hash <br/>

**Verified**. Identifier returned. <br/>

**Failed**: Specifications note that the identifier should be returned immediately and then wait 5 seconds before doing the hash. Instead, the verifier is not returned for 5 seconds (presumably after the hash is complete).<br/>

 - A GET to /hash should accept a job identifier. It should return the base64 encoded password hash for the corresponding POST request. **Verified**. <br/>

Tested: base command curl -H "application/json" http://127.0.0.1:8088/hash/#jobidentifier after submitting several posts. <br/>

**Verified** the hash was consistent by POSTing the same password 3 times and GETting the same result for each request. I assume that the password is stored separately to save time in retrieval as a look up could end up in an excessive wait for an end user. However since the specifications are unclear on this point I would confirm it with the team. <br/>

**Verified** the hash was changing by POSTing a different password and GETting a different result. <br/>

**Verified** the hash was SHA512 base64 by using independent source (https://approsto.com/sha-generator/) <br/>

**Note**: there is a slight variation in the results returned by the program and the results from the independent source, however, since it was consistent across all passwords sent in (== appended) I’m assuming this is just padding added for base 64. <br/>

**Verified** if a non existent job identifier is sent in (ex: 500) that ‘Not found’ is returned. <br/>

**Verified** if no job identifier is sent in that a syntax error occurs. <br/>

 - A GET to /stats should accept no data. It should return a JSON data structure for the total hash requests since the server started and the average time of a hash request in milliseconds. **Partial failure.** <br/>

**Verified** the GET /stats access no data. curl http://127.0.0.1:8088/stats/hash/1 returns 404 page not found. <br/>

**Verified** the data structure is JSON and consists of requests sent and an amount of time. <br/>

**Verified** the number of requests sent was consistent with the operations performed if malformed requests are also included, see notes below. <br/>  

**Failed**. Average time.<br/>
Per the AC the result should be “the average time of a hash request in milliseconds” however my results for 4 or 5 posts were consistently in the hundreds of thousands of milliseconds. Ex: for 4 requests I got 163123 ms = 163.123 seconds = 2.71 minutes which is incorrect because I timed the returns at just about 5 seconds each (small margin of error for stopping the stop watch) so the average for my data set should have been 5 seconds not over 2 minutes. <br/>

I wasn’t able to figure out what the number is tracking, it wasn’t elapsed time and it didn’t appear to be a static value. <br/>

_Interesting to note_: /stats returns the number of requests, even if they were malformed and no hash was created. The specifications are silent on the definition of requests so this may be expected behavior. In a real situation I would point this out to the team to make sure the definition was clear. <br/>

 - The software should be able to process multiple connections simultaneously. **Verified with notation.** <br/>

Tested: Rapidly submitted 5 independent terminal windows with a POST command. Confirmed that all windows returned a unique job identifier and confirmed with the GET that all hashes were unique. <br/>

_Notation_: Five is not necessarily a representative sample size of simultaneous connections.  However, given the limitations of my local machine and the potential issues with a batch process sending requests sequentially I considered it acceptable for this exercise. <br/>

 - The software should support a graceful shutdown request. Meaning, it should allow any in-flight password hashing to complete, reject any new requests, respond with a 200 and shutdown.  And -No additional password requests should be allowed when shutdown is pending. **Verified with notations.** <br/>
 
Tested: Rapidly submitted 10 independent terminal windows. 9 windows had the POST command and the 10th had the shutdown. <br/> 

**Verified** that all processes completed with job identifier returned before the “Shutting down” message appeared in the window executing the code. <br/> 

**Verified** that any new requests were rejected with a failed to connect message. <br/>

_Notation A:_ The typo on the shutdown signal “Shutdown signal recieved”, (recieved instead of received) could cause issues if someone was trying to search for the message. Would recommend fixing. <br/> 

_Notation B:_ I did not receive an explicit 200 response code in any of my testing. However, I am not sure if this is a limitation of the command windows or an actual bug. <br/> 

_Notation C:_ See notes above on sample size limitations. <br/>

 - Issues with the hash program itself: <br/>
 
During exploratory testing I sent in several special characters that produced errors as a result. Since this program has no UI interface to block the submission of special characters my assumption is that it should take any input, therefore I would consider these reportable issues that should be discussed with the team. I also tested excessively long and single digit passwords with no issues. <br/>

1) {“password":"angrymonkey'"} - adding a single quote to the string does not return a job identifier, rather it changes the prompt in the command window to dquote>. <br/>

2) {“password":"ang\y"} - adding a backslash to the string does not return a job identifier, rather it returns Malformed Input. <br/>

3 {“password":"ang”y”} - adding a double quote to the string does not return a job identifier, rather it returns Malformed Input. <br/>

4) {“password":""} - empty string returns a job identifier. When I ran this test I expected it to fail because an empty string seems like an odd thing to accept as a password. <br/>

5) {“password":" "} - blank space returns a job identifier. When I ran this test I expected it to fail because an blank space seems like an odd thing to accept as a password. <br/>
 
