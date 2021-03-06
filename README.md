

Component    | Tools/Packages   | Definition/Usage
-------------| -----------------| -------------
Client (Browser) | React | A JavaScript library for building user interfaces.
Client (Browser) | Apollo Client | A fully-featured caching GraphQL client with integrations for React, Angular, etc - allows you to easily build UI components that fetch data via GraphQL. Helps "bind" GraphQL to our React app to make queries using GraphQL to the server. 
Client (Browser) | GraphiQL                      | A graphical interactive in-browser GraphQL IDE - provides a React component responsible for rendering the UI, which should be provided with a function for fetching from GraphQL.
Server (Node.js) | Express                      | ExpressGraphQL module allows express to understand graphQL and provides a simple way to create an express server that runs the graphQL API - used as middleware on a single route - route acts as endpoint to interact with graphQL endpoint. 
Server (Node.js) | Nodemon                      | Monitors changes in your node.js application and automatically restart the server.
Server (Node.js) | GraphQL | A Query language for APIs and a runtime for fulfilling those queries with your existing data.
Server (Node.js) | Lodash                        | Utility module - a JavaScript library which provides utility functions for common programming tasks using the functional programming paradigm
Database | Mongoose | A MongoDB object modeling tool designed to work in an asynchronous environment - provides a straight-forward, schema-based solution to model your application data. 
Database | mLab | mLab is the largest cloud MongoDB service in the world, hosting over 900000 deployments on AWS, Azure, and Google

----

![Sample Final Site Layout](images/finalImg.png)

# Details on Tools used: :wrench: #

## React ## 
- Create React apps using [Create React App](https://github.com/facebook/create-react-app)

#### React Apollo ####
-  `react-apollo` used to bind apollo to react
- `apollo-boost` used to parse queries from graphQL to JavaScript
- `compose` function imported from react-apollo used to bind multiple queries to a component
- To connect Apollo Client to React, you will need to use the ApolloProvider component exported from react-apollo (imported in `./client/src/App.js`)
  - Apollo provider wraps the application and injects data received from the graphQL server into the application --> do so by wrapping the template in an  `<ApolloProvider>` tag wrapped with the client created (graphQL URI)

## GraphQL ##
 - GraphqlHTTP takes in a schema (object) - which tells express-graphql about the data and how it will look 
 - `export default graphql(getBookDetailsQuery)(BookDetails);` at the bottom of each component .js file is an example of how to bind query to a component. 
    - whenever the component `BookDetails` gets rendered, the particular query `getBooksDetailsQuery` will get run

#### Building the GraphQL Schema ####
- Defining the graph and the object types on the graph
- Root Queries: root queries are graphQLObjects defined in the schema - in fields, you will define everything a client can query for - i.e. Books, authors, etc. in each field, a type is specified (previously defined in schema) and the arguments that can be passed into the query to id the type
- When fields are defined, it's defined as a function because when js is run, it will run top to bottom and if the fields aren't defined as a function, the error "ReferenceError: Type is not defined" will occur. This is because the types may not have been defined at that point in the file. Putting it as a function resolves the catch22, and allows for the file to define all types, and then run the functions (in this case, the fields)
``` 
book(id:'123') {        // argument(s) defined in schema
  name  
  genre
  Author {
    name
  }
}
```

#### GraphQL Types ####
- Cannot simply use primitive types (i.e. string, number, float, etc.) --> use GraphQL specific types

Types               | Definition/Usage
--------------------| -------------
graphQLString       | String 
graphQLID           | IDs can be queried as string or number (increase flexibility) - however, the data must be stored as a STRING
graphQLList         | Returns a list of declared type (i.e.: `type: new GraphQLList(BookType) `)

## GraphiQL ## 
- In-browser tool for writing, validating, and  testing GraphQL queries.
- Added graphiQL to app.js to allow for query testing on to show up when you go to localhost:4000/graphql 
- Documentation explorer: tells you about the graphQL server that you're making queries to - different for each graphQL server. Can be used to QA the various properties/data allowed to be retrieved
- To add new Authors or new Books from the backend, go to `http://localhost:4000/graphql` to directly input queries

## Express & ExpressGraphQL ##
- ExpressGraphQL module allows express to understand graphQL and provides a simple way to create an express server that runs the graphQL API.
- Used as middleware on a single route - route acts as endpoint to interact with graphQL endpoint
- Package `cors` used to allow requests from different servers (in our case, apollo server) 

## MongoDB ##
- Used mLab to spin up an instance of a m ongoDB database (free - used sandbox)

#### Mongoose ####
- Mongoose acts as a front end to MongoDB, an open source NoSQL database that uses a document-oriented data model. A “collection” of “documents”, in a MongoDB database, is analogous to a “table” of “rows” in a relational database

#### Building the MongoDB Schema ####
- The mongoDB/Mongoose schema is different than the graphQL schema - this defines the data that's actually stored inside MongoDB
- When defining a schema, mongoDB automatically defines an ID for each schema 

## Nodemon ##
 - Continuously listens to app.js for changes so restarting server isn't necessary everytime

## Lodash ##
- A JavaScript utility library delivering consistency, mogdularity, performance, & extras - utility module used to help us find data in array in schema.js
- This packaged used to quickly search through dummy data - no longer used after installing and utilizing mongoDB

 `_.find(authors, {id: parent.authorId}) ` : allows you to find in the initial book object (the parent), the author id and the resolve function finds the author and returns that graphQL property to be properly queried 


 - - - -


## Notes on GraphQL Basics :notebook_with_decorative_cover: ## 
Terms/Topics        | Definition/Usage
--------------------| -------------
Mutations | Allows us to mutate or change our data; add, edit, delete data. In graphQL, these need to be explicitly defined to specify what and how data is mutated. 

## Mutations: ##
- Ensure the utilization of `GraphQLNonNull` to prevent inaccurate/inadequate data.
  - In args for each mutation, use `name: { type: new GraphQLNonNull(GraphQLString) }` as opposed to  `name: { type: GraphQLString }`
  - Error message produced when this occurs: `"Field \"addBook\" argument \"genre\" of type \"String!\" is required but not provided." `

Examples Queries:
#### Adding a book by a specific id (id found on the mlab collection) ####
```
mutation {
  addBook(name:"The Color of Magic", genre: "Fantasy" authorId: "5c0de0a2ea35692a65cf3ddc"){
    name
    genre
  }
}
```

#### Nested query for a lit of authors and all the books written by that author ####
```
{
  authors{
    name
    age
    books{
      name
      genre
    }
  }
}
```

### Showing details of a book onced clicked ### 
1. Listen for when a user clicks item
   - In `booklist.js`, add event listener `onClick` to each bullet point (`ul`)
   - Create a constructor that creates a "selected" field that defaults to null and on click, it will set the selected property to the id
2. Find the ID of that book
    - In the `render` function, pass the `bookId` to the booklist component 
3. Pass as `prop` down to the correct component that will render the information
4. Use prop as query variable in the query associated with the component
    - Once attached, in the BookDetails component, when exporting, pass through a second parameter with an "options" property, which will be a function that takes in `props` as a parameter and returns the ID as a variable
      - whenever a new prop comes through (or a new book is clicked), that function will refire, reset and return a new object with a new ID 
from `export default graphql(getBookDetailsQuery)(BookDetails)` to 
  ```
  export default graphql(getBookDetailsQuery, {
    options: function(props) {
      return {
        variables: {
          id: props.bookId
        }
      }
    }
  })(BookDetails);
  ```

 - - - -
 
 ## Styling Notes (CSS) ## 
- All styling for the app in index.css 
- Note that the `id` used in each div/ul/etc. allows you to then style the specific component directly

