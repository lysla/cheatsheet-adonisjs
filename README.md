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