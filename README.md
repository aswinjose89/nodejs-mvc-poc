### Express Model View Controller Pattern (MVC)

The following is an illustration of how we can apply an **MVC** concept to our **NodeJS** application using the **Express Framework**, which later friends can apply when creating an application using Nodejs like Express or others.

#### How to Run:

- install all modules first by typing `npm install` or `yarn add`

- to run it please type `npm run dev` or `yarn run dev`

#### Endpoint Route:

| Name              | Endpoint Route                    |
| ----------------- | --------------------------------- |
| home              | http://localhost:3000             |
| create student  | http://localhost:3000/mhs/create  |
| results student | http://localhost:3000/mhs/results |
| result student  | http://localhost:3000/mhs/result  |
| delete student  | http://localhost:3000/mhs/delete  |
| update student  | http://localhost:3000/mhs/update  |

#### Folder Structure:

- app
- controllers
- helpers
- libs
- middlewares
- models
- routes
- views
- configs
- core
- public

#### Structure:

- **app** a place that contains to store, all the functions of the application that we will make later

- **controller** a place that contains all the logic of the application such as to add student data, delete student data, etc

- **helper** a place that contains a helper function as a utility to use such as **custom message, custom email template** dll

- **libs** a place that contains for customizing libraries that we have installed such as **jwt, bcrypt** which we can later customize into a separate function to use

- **middleware** place containing for custom function middleware used for needs **auth jwt, auth role** dll

- **model** a place that contains for creating schema either with **mongodb or mongoose** which will later be used by **controller** as part of the application logic itself

- **route** a place that contains for the creation of routing in the application to pass functions from **controller to view**

- **config** a place that contains for making configurations from **database** or something else

- **core** controller or core place of application of **model**, **controller**, **route** and **view**

- **public** a place that contains for storing static assets such as **CSS**, **JavaScript**, **Images** etc.

## Here is an example of each function:

#### Core Controller

```javascript
class Controller {
  get(...rest) {
    return router.get(...arguments)
  }
  post(...rest) {
    return router.post(...arguments)
  }
  delete(...rest) {
    return router.delete(...arguments)
  }
  put(...rest) {
    return router.put(...arguments)
  }
}

module.exports = { Controller }
```

#### Core Model

```javascript
class Model {
  constructor(schema) {
    this.model = schema
  }

  findAll(value) {
    const { model, connection } = this
    connection()
    return model.find({ ...value }).lean()
  }
  findOne(value) {
    const { model } = this
    return model.findOne({ ...value }).lean()
  }
  findById(value) {
    const { model } = this
    return model.findById(value).lean()
  }
  findOneAndCreate(value) {
    const { model } = this
    return model.create({ ...value })
  }
  findOneAndDelete(value) {
    const { model } = this
    return model.findOneAndDelete({ ...value }).lean()
  }
  findOneAndUpdate(id, value) {
    const { model } = this
    return model.findOneAndUpdate({ ...id }, { $set: { ...value } }).lean()
  }
}

module.exports = { Model }
```

#### Core Route

```javascript
class Route {
  init() {
    return [
      // init mahasiswa route
      new CreateMahasiswaRoute().route(),
      new ResultsMahasiswaRoute().route(),
      new ResultMahasiswaRoute().route(),
      new DeleteMahasiswaRoute().route(),
      new UpdateMahasiswaRoute().route(),

      //init home route
      new HomeRoute().route(),
      new AboutRoute().route()
    ]
  }
}

module.exports = { Route }
```

#### Core View

```javascript
class View {
  render(res, view, data) {
    res.render(resolve(process.cwd(), `app/views/${view}`), { ...data })
  }
}

module.exports = { View }
```

#### Config Connection

```javascript
class Connection extends Module {
  constructor() {
    super()
    this.db = this.mongoose()
  }
  async MongooseConnection() {
    const { db } = this
    const connection = await db.connect(process.env.MONGO_URI, {
      useUnifiedTopology: true,
      useNewUrlParser: true,
      useFindAndModify: false
    })

    if (!connection) return console.log('Database Connection Failed')
    return console.log('Database Connection Successfuly')
  }
}

module.exports = { Connection }
```

#### Config Module

```javascript
class Module {
  constructor(app) {
    this.app = app
  }
  dotenv() {
    const env = dotenv.config()
    return env
  }
  bodyParser() {
    const { app } = this

    app.use(bodyParser.urlencoded({ extended: false }))
    app.use(bodyParser.json())
  }
  mongoose() {
    mongoose.Promise = global.Promise
    return mongoose
  }
  morgan() {
    const { app } = this
    app.use(logger('dev'))
  }
  event() {
    const events = new EventEmitter()
    return events
  }

  jwt() {
    return jsonwebtoken
  }

  template() {
    const { app } = this
    app.set('views', path.resolve(process.cwd(), 'views'))
    app.set('view engine', 'ejs')
  }

  assets() {
    const { app } = this
    app.use(express.static(path.resolve(process.cwd(), 'public/assets/')))
  }
}

module.exports = { Module }
```

#### App Controller

```javascript
class CreateMahasiswaController extends Model {
  constructor() {
    super()
    this.model = new Model(mhsSchema)
    this.jwt = new Jwt()
  }

  async controller(req, res) {
    const { model, jwt } = this
    const { nama, npm, bid, fak } = req.body
    const user = await model.findOne({ nama, npm, bid, fak })

    if (user) {
      return new CustomeMessage(res).error(409, {
        response: {
          status: 'error',
          code: res.statusCode,
          method: req.method,
          message: 'Oops..data already exists in database',
          data: user
        }
      })
    }

    const { _id } = await model.findOneAndCreate({ nama, npm, bid, fak })
    const token = jwt.createToken({ _id, nama }, { expiresIn: '1d', algorithm: 'HS384' })
    return new CustomeMessage(res).success(200, {
      response: {
        status: 'success',
        code: res.statusCode,
        method: req.method,
        message: 'Yeah..data successuly store in database',
        access_token: token
      }
    })
  }
}

module.exports = { CreateMahasiswaController }
```

#### App Route

```javascript
class CreateMahasiswaRoute extends Controller {
  constructor() {
    super()
  }
  route() {
    return this.post('/mhs/create', (req, res) => new CreateMahasiswaController().controller(req, res))
  }
}

module.exports = { CreateMahasiswaRoute }
```

#### App

```javascript
class App extends Route {
  init() {
    if (cluster.isMaster) {
      let cpuCore = os.cpus().length
      for (let i = 0; i < cpuCore; i++) {
        cluster.fork()
      }
      cluster.on('online', (worker) => {
        if (worker.isConnected()) console.log(`worker is active ${worker.process.pid}`)
      })

      cluster.on('exit', (worker) => {
        if (worker.isDead()) console.log(`worker is dead ${worker.process.pid}`)
        cluster.fork()
      })
    } else {
      //init default route
      app.use(super.init())
      // listenint server port
      http.createServer(app).listen(process.env.PORT)
    }
  }
}

// init application
new App().init()
```
 **NodeJs** and later can apply the concept **MVC** on the application that will be made by friends. **Thank you**
