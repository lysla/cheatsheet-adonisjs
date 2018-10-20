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

## creating a resource

<p>Each resource it's made up of its model, its controller and its migration</p>

```
adonis make:model MyResource --migration --controller
```

<p>🎈 In the created controller, we can remove 'create' and 'edit' method since we are developing just an API and we don't need any form rendering</p>

<p>Then we want to define its Schema, inside the .js migration we just created</p>

```js
class MyResourceSchema extends Schema {
	up () {
		this.create('my_resources', (table) => {
			table.increments()
			table.timestamps()
			// my own resource's fields
			table.string('title')
			table.text('content')
		})
	}

	down () {
		this.drop('my_resources')
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
adonis make:migration MyElement
> Select or Create (if a create already exists)
```

<em>Then update fields and structure in the Schema</em>

```js
class MyElementSchema extends Schema {
	up () {
		this.table('my_elements', (table) => {
			// alter table
			table.dropColumn('content')
		})
	}

	down () {
		this.table('my_elements', (table) => {
			// reverse alternations
			table.text('content')
		})
	}
}
```
# routes

<p>Defining routing for each available data api</p>

```js
Route.resource('my_resources', 'MyResourceController').apiOnly()
```

<p>🎈 this auto generate common routing named as follow</p>

```js
Route.get('my_resources', 'MyResourceController.index').as('my_resources.index')
Route.post('my_resources', 'MyResourceController.store').as('my_resources.store')
Route.get('my_resources/:id', 'MyResourceController.show').as('my_resources.show')
Route.put('my_resources/:id', 'MyResourceController.update').as('my_resources.update')
Route.patch('my_resources/:id', 'MyResourceController.update')
Route.delete('my_resources/:id', 'MyResourceController.destroy').as('my_resources.destroy')
```

# controllers

<p>Controller configuration</p>

<em>Declaration of the const model class</em>

```js
'use strict'
const MyResource = use('App/Models/MyResource')
//...
```

## store method (POST)

<p>Creating a new record</p>

```js
//...
async store ({ request, response }) {

	// retrieving data to store
	const { title, content } = request.post();

	// creating new instance
	const newResource = new MyResource()
	// assigning data
	newResource.title = title
	newResource.content = content

	// save the data to database
	await newResource.save()

	// api response
	response.json({
		message: 'Success',
		data: newResource
	})
}
// ...
```

<em>Alternative</em>

```js
//...
async store ({ request, response }) {

	// retrieving data to store
	const newData = request.only(['title', 'content'])

	// assigning and saving data
	const newResource = await MyResource.create(newData)

	// api response
	response.json({
		message: 'Success',
		data: newResource
	})
}
// ...
```

## index method (GET)

<p>Show all records</p>

```js
async index ({ request, response, view }) {

	// retrieving all data
	const allResources = await MyResource.all()

	// api response
	response.json({
		message: 'Success',
		data: allResources
	})

}
```

## show method (GET)

<p>Show a single record by a given id</p>

```js
async show ({ params, request, response, view }) {

	// retrieve the data by given id
	const theResource = await MyResource.find(params.id)

	// api response
	response.json({
		message: 'Success',
		data: theResource
	})

}
```

## update method (PUT or PATCH)

<p>Edit an existing record by a given id and new data</p>

```js
async update ({ params, request, response }) {

	// retrieve the data by given id
	const theResource = await MyResource.find(params.id)

	// retrieving new data
    const newData = request.only(['title', 'content'])

	// updating data
	theResource.title = newData.title
	theResource.content = newData.content

	// save the data to database
	await theResource.save()

	// api response
	response.json({
		message: 'Success',
		data: theResource
	})

}
```

## destroy method (DELETE)

<p>Remove an existing record by a given id</p>

```js
async destroy ({ params, request, response }) {

	// retrieve the data by given id
	const theResource = await MyResource.find(params.id)

	// delete the record
	await theResource.delete()
	
	// api response
	response.json({
		message: 'Success',
		data: theResource
	})
}
```

# middlewares

<p>Creating a middleware</p>

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

## authentication

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