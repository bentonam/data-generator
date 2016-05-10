# Data Generator

Command-line utility that generates output data in JSON, YAML, CSV based on models which are defined in YAML.  Data can be generated by any combination of [FakerJS](http://marak.github.io/faker.js/), [ChanceJS](http://chancejs.com/), or Custom Build Functions.  Model dependencies can be defined, where data from a previous models generation can be made available to the model currently being generated.   

Generated data can be output in the following formats and destinations:

- JSON files
- YAML files
- CSV files
- Zip Archive of JSON, YAML or CSV files
- Couchbase Server
- Couchbase Sync Gateway Server

## Install

```bash
npm install fakeit -g
```

## Usage

```bash
fakeit [options]
```

## Options

- `-o, --output [value]` *(optional)* The output format to generate.  Supported formats are: json, csv, yaml. The default value is **json**
- `-a, --archive [value]` *(optional)* The archive filename to generate.  Supported formats are: zip.  Example: export.zip
- `-m, --models [value]` *(optional)* A directory or comma-delimited list of files models to use.  The default is the current working directory
- `-d, --destination [value]` *(optional)* The output destination.  Values can be: couchbase, console or a directory path.  The default value is the current working directory.  If the directory path does not exist, it will be created automatically.
- `-f, --format [value]` *(optional)* The spacing format to use for JSON and YAML file generation.  The default value is 2
- `-n, --number [value]` *(optional)* Overrides the number of documents to generate specified by the model
- `-i, --input [value]` *(optional)* A directory of files or a comma-delimited list of files to use as inputs.  Support formats are: json, yaml, csv
- `-s, --server [address]` *(optional)* A Couchbase Server or Sync-Gateway Address.  The default value is **127.0.0.1**
- `-b, --bucket [name]` *(optional)* The name of a Couchbase Bucket.  The default value is **default**
- `-p, --password [value]` *(optional)* A Couchbase Bucket or Sync Gateway user password
- `-g, --sync_gateway_admin [value]` *(optional)* A Sync Gateway Admin address.  
- `-u, --username [value]` *(optional)* A Sync Gateway username.  
- `-h, --help` Displays available options

## Models

All data is generated from one or more [YAML](http://yaml.org/) files.  Models are defined very similar to how models are defined in [Swagger](http://swagger.io/).  With the addition of a few more properties that are used for data generation:

At the root of a model the following keys are used:

- `name:` *(required)* The name of the model
- `type:` *(required)* The data type of the model to be generated
- `key:` *(required)* The main key for the document.  This is a reference to a generated property and is used for the filename or Document ID
- `data:` *(optional)* Defines how many documents should be generated for the model, as well as event callbacks. The following properties are used:
  - `min:` *(optional)* The minimum number of documents to generate
  - `max:` *(optional)* The maximum number of documents to generate
  - `fixed:` *(optional)* A fixed number of documents to generate
  - `pre_run:` *(optional)* A function to be ran before the model generation starts
  - `pre_build:` *(optional)* A function to be ran before each document is generated
  - `post_build:` *(optional)* A function to be ran after each document is generated
  - `post_run:` *(optional)* A function to be ran after all documents for a model have been generated
- `properties:` *(required)* The properties for a model.  Each property can have the following:
  - `type:` *(optional)* The data type of the property.  Values can be: `string`, `number`, `integer`, `long`, `double`, `float`, `array`, `object`
  - `description:` *(optional)* A description of the property
  - `data:` *(optional)* Defines the how the data should be generated.  The following properties can be used:
    - `value:` A static value to be used
    - `fake:` A template string to be used by Faker i.e. `"{{name.firstName}}"`
    - `pre_build:` A function to be called after the value has been initialized.  The property value is assigned from the result.
    - `build:` A function to be called to build the value. The property value is assigned from the result.
    - `post_build:` A function to be called on the property after all of the documents properties have been generated. The property value is assigned from the result.

#### Model Events / Build Functions

Each model can have it's own `pre_(run|build)` and `post_(run|build)` functions, additionally each property can have its on `pre_build`, `build` and `post_build` functions.  

Each one of these functions is passed the following variables that can be used at the time of its execution:

For the `run` functions `this` refers to the current model, for the `build` functions `this` refers to the document currently being generated.

- `documents` - An object containing a key for each model whose value is an array of each document that has been generated
- `globals` - An object containing any global variables that may have been set by any of the run or build functions
- `inputs` - An object containing a key for each input file used whose value is the deserialized version of the files data
- `faker` - A reference to [FakerJS](http://marak.github.io/faker.js/)
- `chance` - A reference to [ChanceJS](http://chancejs.com/)

#### Example users.yaml Model

```yaml
name: Users
type: object
key: _id
data:
  min: 200
  max: 500
  pre_run: >
    globals.user_counter = 0;
properties:
  id:
    type: string
    data:
      post_build: "return 'user_' + this.user_id;"
  type:
    type: string
    data:
      value: "user"
  user_id:
    type: integer
    data:
      build: "return ++globals.user_counter;"
  first_name:
    type: string
    data:
      fake: "{{name.firstName}}"
  last_name:
    type: string
    description: The users last name
    data:
      fake: "{{name.lastName}}"
  email_address:
    type: string
    data:
      fake: "{{internet.email}}"
  phone:
    type: string
    data:
      build: "return chance.phone();"
  created_on:
    type: string
    data:
      fake: "{{date.past}}"
      post_build: "return new Date(this.created_on).toISOString();"
```

We can generate data for this model by executing the following command:

```bash
fakeit -m users.yaml -n 5 -d console
```

This will generate 5 documents for the users model and output the results to the console:

```json
{
  "id": "user_1",
  "type": "user",
  "user_id": 1,
  "first_name": "Emile",
  "last_name": "Murphy",
  "email_address": "Jacques_Langosh0@yahoo.com",
  "phone": "(206) 627-7366",
  "active": true,
  "created_on": "2015-11-20T09:53:33.000Z"
}
{
  "id": "user_2",
  "type": "user",
  "user_id": 2,
  "first_name": "Levi",
  "last_name": "Osinski",
  "email_address": "Franz.Kshlerin@yahoo.com",
  "phone": "(925) 202-9963",
  "active": true,
  "created_on": "2016-04-01T13:54:09.000Z"
}
{
  "id": "user_3",
  "type": "user",
  "user_id": 3,
  "first_name": "Halle",
  "last_name": "Kutch",
  "email_address": "Deontae_Connelly4@gmail.com",
  "phone": "(972) 454-7846",
  "active": true,
  "created_on": "2016-02-28T06:45:42.000Z"
}
{
  "id": "user_4",
  "type": "user",
  "user_id": 4,
  "first_name": "Charlotte",
  "last_name": "Koch",
  "email_address": "Nora.Bauch68@hotmail.com",
  "phone": "(889) 304-9408",
  "active": false,
  "created_on": "2015-07-19T13:49:51.000Z"
}
{
  "id": "user_5",
  "type": "user",
  "user_id": 5,
  "first_name": "Sharon",
  "last_name": "Kutch",
  "email_address": "Jackie.Cremin@gmail.com",
  "phone": "(617) 245-7547",
  "active": true,
  "created_on": "2015-07-20T17:00:51.000Z"
}
```

### Model Dependencies

Often times generated data depends on other generated data, however for this to happen models need to be executed in a certain order.  Lets say we need to generate ecommerce related data and we have the following models:

- orders.yaml (needs data from both products and users)
- products.yaml
- users.yaml

If we were to execute the following to generate the data for these 3 models:

```bash
fakeit -d output/
```

This would fail because By default all models in a directory are used, and will be executed in the order that they are found, which in this case would be:

- orders.yaml
- products.yaml
- users.yaml

The **orders** model needs to reference generated documents from both **products** and **users**.  We could use the `-m` option to ensure that models are executed in order:

```bash
fakeit -d output/ -m users.yaml,products.yaml,orders.yaml
```

While this works, it would require you to remember this order and specify it anytime you would to regenerate the models.  A better approach would be to define the dependencies as part of the model definitions as seen in the [ecommerce example](examples/ecommerce):

**[orders.yaml](examples/ecommerce/orders.yaml)**

```yaml
name: Orders
type: object
key: _id
data:
  dependencies:
    - Products
    - Users
...
```

**[products.yaml](examples/ecommerce/products.yaml)**

```yaml
name: Products
type: object
key: _id
...
```

**[users.yaml](examples/ecommerce/users.yaml)**

```yaml
name: Users
type: object
key: _id
...
```

Now if we execute our original command:

```bash
fakeit -d output/
```

All of the model dependencies will be resolved and executed in the order that satisfies each model dependencies, in this case the model order will be:

- products.yaml
- users.yaml
- orders.yaml

When the **orders.yaml** model is executed the previously generated documents would be made available to the models `run` and `build` functions through the `documents` variable.

```js
documents.Users = [...]; // the name of the users.yaml model
documents.Products = [...]; // the name of the products.yaml model
```

### Model References

It can be beneficial to define definitions that can be referenced one or more times throughout a model.  This can be accomplished by using the `$ref:` property.  Consider the following example:

**[contacts.yaml](examples/contacts/contacts.yaml)**

```yaml
name: Contacts
type: object
key: _id
data:
  min: 1
  max: 4
properties:
  contact_id:
    data:
      build: "return chance.guid();"
  details:
    schema:
      $ref: '#/definitions/Details'
  phones:
    type: array
    items:
      $ref: '#/definitions/Phone'
      data:
        min: 1
        max: 4
  emails:
    type: array
    items:
      $ref: '#/definitions/Email'
      data:
        min: 0
        max: 3
  addresses:
    type: array
    items:
      $ref: '#/definitions/Address'
      data:
        min: 0
        max: 3
definitions:
  Email:
    data:
      build: "return faker.internet.email();"
  Phone:
    type: object
    properties:
      phone_type:
        data:
          build: "return faker.random.arrayElement(['Home', 'Work', 'Mobile', 'Main', 'Other']);"
      phone_number:
        data:
          build: "return faker.phone.phoneNumber().replace(/x[0-9]+$/, '');"
      extension:
        data:
          build: "return chance.bool({likelihood: 20}) ? '' + chance.integer({min: 1000, max: 9999}) : '';"
  Address:
    type: object
    properties:
      address_type:
        data:
          build: "return faker.random.arrayElement(['Home', 'Work', 'Other']);"
      address_1:
        data:
          build: "return faker.address.streetAddress() + ' ' + faker.address.streetSuffix();"
      address_2:
        data:
          build: "return chance.bool({likelihood: 35}) ? faker.address.secondaryAddress() : '';"
      city:
        data:
          build: "return faker.address.city();"
      state:
        data:
          build: "return faker.address.stateAbbr();"
      postal_code:
        data:
          build: "return faker.address.zipCode();"
      country:
        data:
          build: "return faker.address.countryCode();"
  Details:
    type: object
    properties:
      first_name:
        data:
          fake: "{{name.firstName}}"
      last_name:
        data:
          build: "return chance.bool({likelihood: 70})  ? faker.name.lastName() : '';"
      company:
        type: string
        description: The contacts company
        data:
          build: "return chance.bool({likelihood: 30})  ? faker.company.companyName() : '';"
      job_title:
        type: string
        description: The contacts job_title
        data:
          build: "return chance.bool({likelihood: 30})  ? faker.name.jobTitle() : '';"
```

For this model we used 4 references:

- `$ref: '#/definitions/Details'`
- `$ref: '#/definitions/Phone'`
- `$ref: '#/definitions/Email'`
- `$ref: '#/definitions/Address'`

These could have been defined inline but that would make it more difficult to see our model definition and each of these definitions can be reused.  References are processed and included before a model is ran and its documents are generated.

## Examples

The following examples have been provided:

- [ecommerce](examples/ecommerce)
- [contacts](examples/contacts)
- [flat](examples/flat)
- [music](examples/music)
- [simple](examples/simple)

Generate JSON files and output them to a `data/` directory

```bash
[~]$ cd examples/contacts
[~/examples/contacts]$ fakeit -d data/
[~/examples/contacts]$ ls | awk '{print "\011",$NF}'                                       
	 contact_2040b03c-db18-5c85-95b0-37544a2804b1.json
	 contact_239c719f-950e-5411-afd8-dd964c3446bf.json
	 contact_3974b731-792b-5cdd-adc0-0fb56f125dfd.json
	 contact_56284d21-a10f-59bd-8a11-0016329098bd.json
	 contact_8c43f9a6-adcf-5baa-ae78-dd02b8c60944.json
```

---

Generate JSON files and output them to a zip file

```bash
[~]$ cd examples/ecommerce
[~/examples/ecommerce]$ fakeit -a export.zip
[~/examples/ecommerce]$ ls | awk '{print "\011",$NF}'
	 export.zip
	 orders.yaml
	 products.yaml
	 reviews.yaml
	 users.yaml
```

---

Generate JSON files, passing input data, then output them to a zip file in an output directory

```bash
[~]$ fakeit -m ~/examples/music -i ~/examples/music/input -d ~/examples/music/output -a music.zip
[~]$ ls ~/examples/music/output | awk '{print "\011",$NF}'
	 music.zip
```

---

Generate CSV files for each model and output them to a directory

```bash
[~]$ fakeit -o csv -m ~/examples/flat -d ~/csv
[~]$ ls ~/csv | awk '{print "\011",$NF}'
	 Products.csv
	 Users.csv
```

---

Generate CSV files for each model and save them to a zip file

```bash
[~]$ fakeit -o csv -m ~/examples/flat -d ~/csv -a export.zip
[~]$ ls ~/csv | awk '{print "\011",$NF}'
	 export.zip
```

---

Generate JSON documents and output them to Couchbase using the defaults of a server running at `127.0.0.1` in the bucket `default`

```bash
[~]$ fakeit -d couchbase -m ~/examples/music
```

---

Generate JSON documents and output them to Couchbase Server running at `192.168.1.101` in the bucket `music` with the password of `secret`

```bash
[~]$ fakeit -d couchbase -m ~/examples/music -s 192.168.1.101 -b music -p secret
```

---

Generate JSON documents and output them to a Couchbase Sync Gateway running at `localhost` into a `contacts` bucket with guest access enabled.

```bash
[~]$ fakeit -d sync-gateway -m ~/examples/contacts -s http://localhost:4984 -b contacts
```

---

Generate JSON documents and output them to a Couchbase Sync Gateway running at `localhost:4984` into a `contacts` bucket using [Custom (Indirect) Authentication](http://developer.couchbase.com/documentation/mobile/current/develop/guides/sync-gateway/administering-sync-gateway/authenticating-users/index.html).

```bash
[~]$ fakeit -d sync-gateway -m ~/examples/contacts -s http://localhost:4984 -b contacts -g http://localhost:4985 -u jdoe -p supersecret
```