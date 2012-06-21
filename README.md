#Planworld v3 Client API

This API allows you to write alternative user-facing interfaces for planworld. It is not
appropriate for server-server communication, which will be covered later in a v3 Server
API revision.

For reference:  
* [HTTP Verbs](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
* [HTTP Status Codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

##Before you begin
Some principles to keep in mind:

1. CLIENT KEY REQUIRED  
   All clients need a Client Key, which you can get from any of the planworld nodes.
   If an API request does not include a client key, the server must respond with a 
   449 Retry with valid parameters status and explanatory text.

2. USER AGENT REQUIRED  
   Clients must report the user agent used to register for their API key when making
   requests. 

3. SSL REQUIRED  
   Clients must use SSL for all communication. If an API request is sent unencrypted,
   the server must respond with a 426 Upgrade Required status and explanatory text.

4. USER KEY SUGGESTED  
   Clients are strongly encouraged to use the User Key flow, detailed below, when possible

5. SECURE CREDENTIAL STORAGE REQUIRED  
   If a client stores usernames and passwords for authentication, they must be encrypted
   and stored securely using the best available means. Client Key revocation will result
   for clients that do not safeguard user credentials.

6. API ENDPOINTS MAY CHANGE  
   Clients should include the API endpoint of the server that issued their Client Key
   for distribution, and call /nodes periodically to learn about other nodes and
   endpoints. Servers should provide an overlap of at least one week when changing
   API endpoints to give all clients time for discovery.
   
7. TOKENS WILL EXPIRE  
   Clients are encouraged to refresh/replace tokens before expiry. Keep in mind that 
   clock skew may come into play, and give yourself a buffer of at least a few seconds.
   
8. RESPONSE FORMAT  
   Clients may specify the response format as an extension in the URL. The server should
   return JSON if no format is specified.


##Endpoint URLs
Endpoint URLs are omitted from the API requests below. A complete API request looks
like this:

https://server.com/endpoint/ _api/request_.format ?clientkey= _client key_ &userkey = _user key_  
or  
https://server.com/endpoint/ _api/request_.format ?clientkey= _client key_ &token = _user token_

For API calls that require different query string parameters (the stuff after the ?) they
will be specified. You will always need to include the client key and a user identifier.

##Example Flows ############

### Authentication / Authorization ##

#### Getting a User Key #####
This is oauth-ish.

1. Prompt the user for a username but no password. Mark the submit button "Authorize" or
   similar if you're presenting a GUI.
2. Redirect the user to their node in a browser (or a web view if you must) like so:
   GET /authorize/username?callback=YOUR_CALLBACK_URL
3. The user will be presented with a login form, or if already logged in...
4. A logged in user will be presented with an option to allow or deny access to your client.
5. The user will then be redirected to the callback URL provided by your client with a
   USER_KEY parameter. You may store this, and the user may revoke it through the 
   web interface. You may also provide an interface to revoke User Keys if you like.
   

#### Getting a User Token ###
GET /token  
?clientkey= _your client key_ &username= _the username_ &password= _the user password_

SUCCESS:  
* 200 OK with the token as the body  

FAILURE:  
* 401 Unauthorized, error text may be included in the body

### Plans ###

#### Getting a Plan or Entry #########
GET /plan/_planname_  
GET /plan/_planname_/entry/_entryid_  

SUCCESS:  
* 200 OK, plan content as the body  

FAILURE:  
* 404 Not Found if the user has no plan  
* 403 Forbidden if the reader is not allowed  
* 400 Bad Request if the user does not exist  

notes:   
* 	use the accept header to indicate the desired plan formatting.
	(plaintext, html, unprocessed)

* 	a server may respond with 400 Bad Request for specifically blocked readers to protect
		user privacy, but should always use 403 Forbidden for nonspecific blocking such as
		crossnode bans.


#### Updating a Plan or entry ########
POST /plan/_planname_  
POST /plan/_planname_/entry/_entryid_  

POST DATA: entry= _plan entry text_ & parameters= _plan metadata struct_

SUCCESS:    
* 201 Created for a journaling plan  
* 200 OK for a traditional plan  

FAILURE:  
* 403 Forbidden if the writer cannot update that plan

notes:  
* 	clients should GET /username/plan/settings before presenting the plan update form
		so that you can provide the appropriate parameters  

* 	if the plan is traditional-style, clients should get the content of the current plan
		(with an accept header "unprocessed") to present in the update form.  

* 	clients should GET /plan/_planname_ on response to refresh the view


#### Removing a Plan or Entry ########
DELETE /plan/_planname_  
DELETE /plan/_planname_/entry/_entryid_  

SUCCESS:  
* 205 Reset Content  

FAILURE:   
* 403 Forbidden if the writer cannot update that plan  
* 404 Not Found if the plan or entry does not exist  

notes:  
* 	clients should GET /plan/_planname_ on response to refresh the view  


### User Info / Whois ##
#### Getting User Info ######
GET /info/_username_

SUCCESS:  
* 200 OK, bio or other whois content as the body  

FAILURE:  
* 404 Not Found if the user has no bio/info stored  
* 403 Forbidden if the reader is not allowed  
* 400 Bad Request if the user does not exist  

notes:   
* 	use the accept header to indicate the desired plan formatting.
	(plaintext, html, unprocessed)

* 	a server may respond with 400 Bad Request for specifically blocked readers to protect
		user privacy, but should always use 403 Forbidden for nonspecific blocking such as
		crossnode bans.


### Watched List ###

#### Getting a Watched List ########
GET /planwatch

SUCCESS:
* 200 OK and the watched list struct

notes:  
* documentation of watched list structure to follow