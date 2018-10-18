<h1>lysla's API with AdonisJS cheatsheet</h1>
<p>Simple indexed manual with notes.</p>

# installation

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

## relate multiple resources (1 to N)

<p>We use a foreign key to relate one resource record to one or more some other resource's records</p>

```js
class MyElementSchema extends Schema {
	up () {
		this.create('my_elements', (table) => {
			table.increments()
			table.timestamps()
			// my own element's fields
			table.string('title')
			table.text('content')
			// foreign key to relate N records to a specific resource record
			table.integer('fk_resource').unsigned()
			table
			.foreign('fk_resource')
			.references('my_resources.id')
			.onDelete('cascade')
		})
	}

	down () {
		this.drop('my_resources')
	}
}
```

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