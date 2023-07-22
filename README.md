# Handles Tracking with Trac
This document is supposed to help retrieving indexed & tracked handles data for further processing.

After public release, you may want to self-host Trac. In this case, the described endpoint below will need to get changed into your own endpoint location.

The examples below use websockets instead of classic RESTful API endpoints as these usually serve large amounts of data better. With the release of Trac, RESTful APIs will be available but should only be used for smaller sets of data.

#### Requirements
- Some Javascript knowledge (Node or Browser)
- Socket.io 4.6.1

#### Setup

HTML/JS

```javascript
<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.6.1/socket.io.min.js" integrity="sha512-AI5A3zIoeRSEEX9z3Vyir8NqSMC1pY7r5h2cE+9J6FLsoEmSSGLFaqMQw8SWvoONXogkfFrkQiJfLeHLz3+HOg==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
```

Node

```npm install socket.io-client```

Code Anatomy

```javascript
// node only!
const { io } = require("socket.io-client");

// connect to public handles endpoint with Trac

const trac = io("https://handles.trac.network", {
    autoConnect : true,
    reconnection: true,
    reconnectionDelay: 500,
    econnectionDelayMax : 500,
    randomizationFactor : 0
});

trac.connect();

// default response event for all endpoint calls.
// this event handles all incoming results for requests performed using "emit".
//
// You may use the response to render client updates or store in your local database cache.
//
// Example response object: 
// {
//    "error": "",
//    "func": ORIGINALLY-CALLED-FUNCTION-NAME,
//    "args": ARRAY-WITH-ARGUMENTS,
//    "call_id": OPTIONAL-CUSTOM-VALUE,
//    "result": RETURNED-MIXED-TYPE-VALUE
// }

trac.on('response', async function(msg){
  console.log(msg);
});

// default error event for internal Trac errors, if any

trac.on('error', async function(msg){
    console.log(msg);
});

// example getter to get the transaction size of a block
// the results for this call will be triggered by the 'response' event above.
// this structure is exactly the same for all available getters of this endpoint.

trac.emit('get',
{
    func : 'handle',     // the endpoint's function to call
    args : ['@handle'],  // the arguments for the function (in this case only 1 argument, the block)
    call_id : ''         // a custom id that is passed through in the 'response' event above to identify for which call the response has been.
});
```

#### Available endpoint getters

The results for the calls of the below getters should be used in the "response" event above, explained in Code Anatomy.

Parallel calls are encouraged. Especially if large amounts of data are requested, it is recommended to request them in parallel and smaller chunks.

```javascript

/**
* Accepts a handle string to lookup.
* Returns an object with inscription id, inscription number and original handle as entered by the inscriber.
* Returns null if the handle doesn't exist.
*/

trac.emit('get',
{
    func : 'handle',
    args : [handle],
    call_id : ''
});

/**
* Returns the size of valid tracker handles.
* Can be used with "handleByIndex" to iterater over the entire index.
*/

trac.emit('get',
{
    func : 'handlesLength',
    args : [],
    call_id : ''
});

/**
* Accepts an integer as position of the tracked handle.
* Returns an object with inscription id, inscription number and original handle as entered by the inscriber.
* Returns null if the handle doesn't exist.
*
* Can be used alongside 'handlesLength' to iterater over the entire handles index.
* Order is not guaranteed. Please use the inscription numbers to sort locally in your cache after pulling the index.
*/

trac.emit('get',
{
    func : 'handleByIndex',
    args : [index],
    call_id : ''
});

/**
* Returns an ordinal object with base64-encoded content, mime type, blocks, timestamp and more.
*/

trac.emit('get',
{
    func : 'ordinal',
    args : [inscription id or number],
    call_id : ''
});

/**
* Returns that last known block number.
*/

trac.emit('get',
{
    func : 'latestBlock',
    args : [],
    call_id : ''
});

/**
 * Returns an object with owner address, block and tx hash of the given inscription id.
 * Returns null if not found.
 */

trac.emit('get',
{
    func : 'ownerOf',
    args : [inscription id],
    call_id : ''
});
```
