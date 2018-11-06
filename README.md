<h1>lysla's API with AdonisJS cheatsheet</h1>
<p>Simple indexed manual with notes.</p>

# installation

```
npm i -g @adonisjs/cli
```

<em><small>To execute in project directory</small></em>
```
adonis new . --api-only
```
<em><small>Then start it</small></em>

```
adonis serve --dev
```

## configuring database connection

<p>MySQL example (on localhost)</p>

```
npm i --save mysql
```

<em><small>In config/database.js</small></em>
```js
//...
connection: Env.get('DB_CONNECTION', 'mysql')
//...
mysql: {
    client: 'mysql',
    connection: {
      host: Env.get('DB_HOST', 'localhost'),
      port: Env.get('DB_PORT', ''),
      user: Env.get('DB_USER', 'root'),
      password: Env.get('DB_PASSWORD', ''),
      database: Env.get('DB_DATABASE', 'lysla-cms-api')
    },
    debug: Env.get('DB_DEBUG', false)
}
```

<em><small>In .env file</small></em>
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=
DB_DATABASE=lysla-cms-api
```

## creating a resource

<p>Each resource it's made up of its model, its controller and its migration</p>

```
adonis make:model Item --migration --controller
```

<p>🎈 In the created controller, we can remove 'create' and 'edit' method since we are developing just an API and we don't need any form rendering</p>

<p>Then we want to define its schema via the migration we just created</p>

```js
class ItemSchema extends Schema {
	up () {
		this.create('items', (table) => {
			table.increments()
			table.timestamps()
			// my own resource's fields
			table.string('name')
			table.text('content')
		})
	}

	down () {
		this.drop('items')
	}
}
```

<p>🎈 Remind we got to create and run proper database structure for all kind of relationships (foreign keys and pivot table for many to many) manually with migrations</p>

# migrations 

<em><small>Run migrations to populate, or update, database</small></em>

```
adonis migration:run
```

<em><small>Or roll em back</small></em>
```
adonis migration:rollback
```

<em>Create a migration for each schema change</em>

```
adonis make:migration Item
> Select or Create (if a create already exists)
```

<em>Then update fields and structure in the Schema</em>

```js
class ItemSchema extends Schema {
	up () {
		this.table('items', (table) => {
			// alter table
			table.dropColumn('name')
		})
	}

	down () {
		this.table('items', (table) => {
			// reverse alternations
			table.text('name')
		})
	}
}
```

<p>🎈 Remind that Adonis migration syntax is developed on the top of knexjs.org whom enable us to define migration for tables with relationship via foreign keys</p>

## foreign key

<p>Migration with foreign keys</p>

```js
class CardSchema extends Schema {
	up() {
		this.create('cards', (table) => {
			table.increments()
			table.timestamps()

			table.string('brand')
			table.string('weight')
			// foreign key to relate to items table
			table.integer('item_id').unsigned()
			table.foreign('item_id').references('items.id')
		})
	}

	down() {
		this.drop('cards')
	}
}
```

<em><small>We can also add a foreign key to an existing table with a new migration</small></em>

```js
class CardSchema extends Schema {
	up() {
		this.table('cards', (table) => {
			table.integer('item_id').unsigned()
			table.foreign('item_id').references('items.id')
		})
	}

	down() {
		this.table('cards', (table) => {			
		 	table.dropForeign('item_id')
		})
	}
}
```

## pivot table

<p>Migration for pivot table</p>

```
adonis make:migration item_tag
> Create table
```

```js
class ItemTagSchema extends Schema {
	up() {
		this.create('item_tag', (table) => {
			table.integer('item_id').unsigned()
			table.foreign('item_id').references('items.id')
			table.integer('tag_id').unsigned()
			table.foreign('tag_id').references('tags.id')
		})
	}

	down() {
		this.drop('item_tag')
	}
}
```

<p>Migration for pivot model</p>

```
adonis make:migration item_box
> Create table
```

```js
class ItemBoxSchema extends Schema {
	up() {
		this.create('item_box', (table) => {
			table.increments()
			table.timestamps()

			table.integer('item_id').unsigned()
			table.foreign('item_id').references('items.id')
			table.integer('box_id').unsigned()
			table.foreign('box_id').references('boxes.id')

			table.integer('quantity')			
		})
	}

	down() {
		this.drop('item_box')
	}
}
```

# models (💥 wip)

<p>Each model mediates under the hood for every normal table field. Getters and setters will override its default process, as long as they respect the naming constraint.</p>

<em>A field named 'title' will need a getter 'getTitle' and a setter 'setTitle' (camecase)</em>

```js
//example 💥wip
```

<p>More then getters and setters, models allows computed properties</p>

```js
//example 💥wip
```

## relationships

<p>Where the Adonis models works as ORM we always need to create proper migrations manually for each relationship, following standard RDBMS school.</p>

### one to one - 1:1

<p>Where every Item has one and only one Card</p>


```js
class Item extends Model {
	card () {
		return this.hasOne('App/Models/Card')
	}
}
```

### one to many - 1:n

<p>Where every Category can contain one or more Item(s)</p>

```js
class Category extends Model {
	items () {
		return this.hasMany('App/Models/Item')
	}
}
```

### many to one - n:1

<p>Where every Item(s) belongs to one and only one Category</p>

```js
class Item extends Model {
	category () {
		return this.belongsTo('App/Models/Category')
	}
}
```

### many to many - n:m

<p>Where every Item(s) can belong to one or more Tag(s)</p>

```js
class Item extends Model {
	tags () {
		return this.belongsToMany('App/Models/Tag')
	}
}
```
```js
class Tag extends Model {
	items () {
		return this.belongsToMany('App/Models/Item')
	}
}
```

#### pivot model

<p>With pivot models we can save and access more data throu the pivot table relation</p>

<p>🎈 Given we already created the proper pivot table migration, we can create its model</p>

```
adonis make:model ItemBox
```
<p>And then define the many to many relation in the model</p>

```js
class Box extends Model {
	items () {
		return this
			.belongsToMany('App/Models/Item')
			.pivotModel('App/Models/ItemBox')
	}
}
```
```js
class Item extends Model {
	boxes () {
		return this
			.belongsToMany('App/Models/Box')
			.pivotModel('App/Models/ItemBox')
	}
}
```


# routes

<p>Defining routing for each available data api</p>
<em><small>In start/routes.js</small></em>

```js
Route.resource('items', 'ItemController').apiOnly()
```

<p>🎈 this auto generate common routing named as follow</p>

```js
Route.get('items', 'ItemController.index').as('items.index')
Route.post('items', 'ItemController.store').as('items.store')
Route.get('items/:id', 'ItemController.show').as('items.show')
Route.put('items/:id', 'ItemController.update').as('items.update')
Route.patch('items/:id', 'ItemController.update')
Route.delete('items/:id', 'ItemController.destroy').as('items.destroy')
```

# controllers

<p>Controller configuration</p>

<em>Remind the declarations of const model class</em>

```js
'use strict'
const Item = use('App/Models/Item')
//...
```

## index method (GET)

<p>Show all records</p>

```js
async index ({ request, response, view }) {

	// retrieving all data
	const allItems = await Item.all()

	// api response
	response.json({
		message: 'Success',
		data: allItems
	})

}
```

## store method (POST)

<p>Creating a new record</p>

```js
//...
async store ({ request, response }) {

	// retrieving data to store
	const { name, content } = request.all();

	// creating new instance
	const newItem = new Item()
	// assigning data
	newItem.name = name
	newItem.content = content

	// save the data to database
	await newItem.save()

	// api response
	response.json({
		message: 'Success',
		data: newItem
	})
}
// ...
```

<em>Alternative with selective income values</em>

```js
//...
async store ({ request, response }) {

	// retrieving data to store
	const newData = request.only(['title', 'content'])

	// assigning and saving data
	const newItem = await Item.create(newData)

	// api response
	response.json({
		message: 'Success',
		data: newItem
	})
}
// ...
```

## show method (GET)

<p>Show a single record by a given id</p>

```js
async show ({ params, request, response, view }) {

	// retrieve the data by given id
	const theItem = await Item.find(params.id)

	// api response
	response.json({
		message: 'Success',
		data: theItem
	})

}
```

## update method (PUT or PATCH)

<p>Edit an existing record by a given id and new data</p>

```js
async update ({ params, request, response }) {

	// retrieve the data by given id
	const theItem = await Item.find(params.id)

	// retrieving new data
    const newData = request.only(['name', 'content'])

	// updating data
	theItem.name = newData.name
	theItem.content = newData.content

	// save the data to database
	await theItem.save()

	// api response
	response.json({
		message: 'Success',
		data: theItem
	})

}
```

## destroy method (DELETE)

<p>Remove an existing record by a given id</p>

```js
async destroy ({ params, request, response }) {

	// retrieve the data by given id
	const theItem = await Item.find(params.id)

	// delete the record
	await theItem.delete()
	
	// api response
	response.json({
		message: 'Success',
		data: theItem
	})
}
```

# querying

<p>Querying complex data throu modeled relationships</p>
<p>💥 work in progress</p>

# middlewares

<p>Creating a middleware</p>
<p>🎈 Adonis apiOnly boilerplate comes with authentication middlewares out of the box so we don't need to create them, the following it's just an example for a custom middleware</p>

```
adonis make:middleware MyMiddleware
> HTTP Requests
```

<em><small>Then in kernel.js we need to register the middleware</small></em>

```js
// as named middleware (only for some routes)
const namedMiddleware = {
	//...
	myMiddleware: 'App/Middleware/My'
}
```

<em><small>Then in the routes.js, in the specific route you want to plate the middleware within</small></em>

```js
Route.get('my_resources/:id', 'MyResourceController.show').as('my_resources.show').middleware(['myMiddleware'])
```

<em><small>Or using resource that autogenerates routes</small></em>

```js
Route.resource('my_resources', 'MyResourceController')
	.apiOnly()
	.middleware(new Map([
		[['show','update','destroy'], ['myMiddleware']]
	]))
```

<em><small>Then in the middleware file</small></em>

```js
async handle ({ request }, next) {
	
	// do somethign with the request before advancing
	console.log('My Middleware Fired')

	// call next to advance the request
	await next()
}
```

## authentication (💥 wip)

<p>Adonis provide JWT authentication out of the box, with 'auth' and 'guest' middleware</p>

<em><small>Configure routing for login and guard others</small></em>
```js
Route
  .post('login', 'UserController.login')
  .middleware('guest')

Route.resource('my_resources', 'MyResourceController')
	.apiOnly()
	.middleware(['auth'])
```

<p>Create and configure User controller (model and starter migration are out of the box)</p>

```
adonis make:controller User
> For HTTP requests
```

<em><small>Then in the user controller</small></em>

```js
class UserController {

	async login ({ request, response, auth }) {
		const { email, password } = request.only(['email', 'password'])
		
		const jwt = await auth.attempt(email, password)

		// api response
		response.json({
			message: 'Success',
			data: jwt
		})
	}
}
```

<p>Like so, we receive a JWT token that will need to be passed throu Headers in requests to get authorization for guarded routes</p>

```
In Headers request:
Authorization = Bearer <mytoken>
```

<p>🎈 There is no manual logout with JWT authentication, but we can configure token expiration time</p>

<em><small>In auth.js</small></em>

```js
jwt: {
	serializer: 'lucid',
	model: 'App/Models/User',
	scheme: 'jwt',
	uid: 'email',
	password: 'password',
	options: {
		secret: Env.get('APP_KEY'),
		// custom token duration (in seconds)
		expiresIn: 60
	}
}
```