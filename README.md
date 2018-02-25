# graphql-aql-generator
schema graphql query – query GraphQL with only a GraphQL IDL schema.

Are you tired of writing GraphQL.js schema files?

Then try `graphql-aql-generator`. `graphql-aql-generator` needs only a GraphQL IDL file and generates the GraphQL.js schema automatically.



# Example


```es6
'use strict';

const graphql = require('graphql-sync').graphql;
const generator = require('graphql-aql-generator');


let typeDefs = [`
  type BlogEntry {
    _key: String!
    authorKey: String!

    author: Author @aql(exec: "FOR author in Author filter author._key == @current.authorKey return author")
  }

  type Author {
    _key: String!
    name: String
  }

  type Query {
    blogEntry(_key: String!): BlogEntry
  }
`]

const schema = generator(typeDefs);

const query = `
{
  blogEntry(_key: "1") {
    _key
    authorKey
    author {
      name
     }
  }
}`;

const result = graphql(schema, query);
print(result);
/*
{
  "data" : {
    "blogEntry" : {
      "_key" : "1",
      "authorKey" : "2",
      "author" : {
        "name" : "Plumbum"
      }
    }
  }
}
*/`


// or use it with a GraphQLRouter

'use strict';

const createRouter = require('@arangodb/foxx/router');
const router = createRouter();
module.context.use(router);

const createGraphQLRouter = require('@arangodb/foxx/graphql');

const graphql = require('graphql-sync').graphql;
const sgq = require('sgq');

const typeDefs = [`
  type BlogEntry {
    _key: String!
    authorKey: String!

    author: Author @aql(exec: "FOR author in Author filter author._key == @current.authorKey return author")
  }

  type Author {
    _key: String!
    name: String
  }

  type Query {
    blogEntry(_key: String!): BlogEntry
  }
`]

const schema = sgq(typeDefs);

router.use('/graphql', createGraphQLRouter({
  schema: schema,
  graphiql: true,
  graphql: require('graphql-sync')
}));

```
In this example two types and a query are defined. `BlogEntry` and `Author`. BlogEntry has a sub attribute Author which is fetched with a @aql directive. The query returns a BlogEntry with the corresponding Author depending on the BlogEntries _key.

in order to use remote debugger via node express (branch: for-remote-debug):
const express = require("express");
const generator = require("graphql-aql-generator");
const handelr = require("graphql-aql-generator/handler");
const arangojs = require("arangojs-debug");
const graphql-sync = require("graphql-sync")

const db = arangojs({
  url: `http://${username}:${password}@${host}:${port}`
});

const arangoaSchema = generator(arangoDBtypeDefs, db);

// Initialize the app
const app = express();

// The GraphQL endpoint
app.use(
  "/graphql",
  bodyParser.json(),
  handelr({
    schema: arangoaSchema,
    graphiql: true,
    graphql: graphql-sync
  })
);

// Start the server
app.listen(3000, () => {
  console.log("Go to http://localhost:3000/graphiql to run queries!");
});
