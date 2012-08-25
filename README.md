#[Patio](http://pollenware.github.com/patio)

Patio is a <a href="http://sequel.rubyforge.org/" target="patioapi">Sequel</a> inspired query engine.                                                        
                                                                                                                                                             
###Why Use Patio?
                                                                                                                                                             
Patio is different because it allows the developers to choose the level of abtraction they are comfortable with.                                             

If you want to use [ORM](http://pollenware.github.com/patio/models.html) functionality you can. If you dont you can just use the [Database](http://pollenware.github.com/patio/DDL.html) and [Datasets](http://pollenware.github.com/patio/querying.html) as a querying API, and if you need toyou can [write plain SQL](http://pollenware.github.com/patio/patio_Database.html#run)
                                                                                                                                                                                                                                                                                                                         
###Installation
To install patio run                                                                                                                                         
                                                                                                                                                             
`npm install comb patio`
                                                                                                                                                             
If you want to use the patio executable for migrations
                                                                                                                                                             
`npm install -g patio`
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
###Getting Started  


Create some tables.
                                                                                                                      
```                                                                                                                                    
var patio = require("patio"),                                                                                                    
     comb = require("comb"),                                                                                                           
     when = comb.when,                                                                                                                 
     serial = comb.serial;                                                                                                             
                                                                                                                                       
                                                                                                                                       
 //set all db name to camelize                                                                                                         
 patio.camelize = true;                                                                                                                
 patio.configureLogging();                                                                                                             
 //connect to the db                                                                                                                   
 var DB = patio.connect(<CONNECTION_URI>);                                                                                       
                                                                                                                                       
function errorHandler(error) {                                                                                                 
     console.log(error);                                                                                                               
     patio.disconnect();                                                                                                               
 };                                                                                                                                    
                                                                                                                                       
function createTables() {
    return comb.serial([
        function () {
            return DB.forceDropTable(["capital", "state"]);
        },
        function () {
            return DB.createTable("state", function () {
                this.primaryKey("id");
                this.name(String)
                this.population("integer");
                this.founded(Date);
                this.climate(String);
                this.description("text");
            });
        },
        function () {
            return DB.createTable("capital", function () {
                this.primaryKey("id");
                this.population("integer");
                this.name(String);
                this.founded(Date);
                this.foreignKey("stateId", "state", {key:"id"});
            });
        }
    ]);
};                                                                                               
                                                                                                                                       
 createTables().then(function () {                                                                                                     
    patio.disconnect();                                                                                                                 
}, errorHandler);                                                                                                                      
```   

Next lets create some models for the tables created.

```
var State = patio.addModel("state", {
    static:{
        init:function () {
            this._super(arguments);
            this.oneToOne("capital");
        }
    }
});
var Capital = patio.addModel("capital", {
    static:{
        init:function () {
            this._super(arguments);
            this.manyToOne("state");
        }
    }
});
```

Next you'll need to sync your models

```
patio.syncModels();
```

Use your models.

```
//comb.when waits for the save operta
return comb.when(
	State.save({
        name:"Nebraska",
        population:1796619,
        founded:new Date(1867, 2, 4),
        climate:"continental",
        capital:{
            name:"Lincoln",
            founded:new Date(1856, 0, 1),
            population:258379
        }
    }),
    Capital.save({
        name:"Austin",
        founded:new Date(1835, 0, 1),
        population:790390,
        state:{
            name:"Texas",
            population:25674681,
            founded:new Date(1845, 11, 29)
        }
    })
);
```

Now we can query the states and capitals we created.

```
State.order("name").forEach(function (state) {
	//if you return a promise here it will prevent the foreach from
	//resolving until all inner processing has finished.
	return state.capital.then(function (capital) {
    	console.log("%s's capital is %s.", state.name, capital.name);
	});
});
```

```
Capital.order("name").forEach(function (capital) {
	//if you return a promise here it will prevent the foreach from
	//resolving until all inner processing has finished.
	return capital.state.then(function (state) {
		console.log(comb.string.format("%s is the capital of %s.", capital.name, state.name));
	});
});
```

###Features
                                                                                                                                                                                                                                                                                                          
* Comprehensive documentation with examples.
* &gt; 80% test coverage
* Support for connection URIs
* Supported Databases                                                                                                                                        
  * MySQL
  * Postgres
* [Models](http://pollenware.github.com/patio/models.html)
  * [Associations](http://pollenware.github.com/patio/associations.html)
  * [Inheritance](http://pollenware.github.com/patio/model-inheritance.html)
  * [Validation](http://pollenware.github.com/patio/validation.html)
  * [Plugins](http://pollenware.github.com/patio/plugins.html)
* Simple adapter extensions
* [Migrations](http://pollenware.github.com/patio/migrations.html)
  * Integer and Timestamp based.
* Powerful [Querying](http://pollenware.github.com/patio/querying.html) API
* [Transactions](http://pollenware.github.com/patio/patio_Database.html#transaction) with
  * Savepoints
  * Isolation Levels
  * Two phase commits
* SQL Datatype casting
* Full database CRUD operations                                                                                                                           
  * [createTable](http://pollenware.github.com/patio/patio_Database.html#createTable)
  * [alterTable](http://pollenware.github.com/patio/patio_Database.html#alterTable)
  * [dropTable](http://pollenware.github.com/patio/patio_Database.html#dropTable)
  * [insert](http://pollenware.github.com/patio/patio_Dataset.html#insert)
  * [multiInsert](http://pollenware.github.com/patio/patio_Dataset.html#multiInsert)
  * [update](http://pollenware.github.com/patio/patio_Dataset.html#update)
  * [remove](http://pollenware.github.com/patio/patio_Dataset.html#remove)
  * [query](http://pollenware.github.com/patio/patio_Dataset.html#filter)



                                                                                                                                   