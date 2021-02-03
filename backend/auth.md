# <kbd>Authentication (Session Based)</kbd>
### Backend
Assuming you have a working REST API, implementing authentication in the backend is a piece of cake. The first thing we have to do is to embed session library to our server.
For express you can use [express-session](https://www.npmjs.com/package/express-session) and [connect-mongo](https://www.npmjs.com/package/connect-mongo).
The point is session can be stored in local memory -may you prefer for development yet not secure for production- or in a database which is the right way for production.
There are a lot of [session stores](https://www.npmjs.com/package/express-session#compatible-session-stores) nearly for every db library.
```js
// server.js
const session = require('express-session')
const mongoose = require('mongoose')
const connectStore = require('connect-mongo')
// Other imports ...

(async () => {
    try {
        // Connect to the database
        await mongoose.connect(MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
        const app = express()
        // For connect-mongo you can use a constructor builder function to get a store from a session
        const MongoStore = connectStore(session)    
        // Use other middlewares
        // For the session properties you can read express-session documentation
        app.use(session({
            name: SESS_NAME,
            secret: SESS_SECRET,
            resave: false,
            saveUninitialized: false,
            store: new MongoStore({
                // You can create another connection for sessions yet it's a bit overhead
                mongooseConnection: mongoose.connection,
                collection: 'session',
                ttl: parseInt(SESS_LIFETIME) / 1000
            }),
            cookie: {
                sameSite: true,
                secure: NODE_ENV === 'production',
                maxAge: parseInt(SESS_LIFETIME)
            }
        }))
        // Inject routers then listen the server
    } catch (err) {
        console.log(err)
    }
})()
```
When we are done with initiation our session store, now we can create a router(or a controller) for our session. The session controller should have three basic request methods:
`post`, `delete` and `get`. Post is for creating a new session, delete is for deleting a session and get is for getting the current session.
