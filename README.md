# Reason-apollo

[![npm version](https://badge.fury.io/js/reason-apollo.svg)](https://badge.fury.io/js/reason-apollo)
[![Get on Slack](https://img.shields.io/badge/slack-join-orange.svg)](http://www.apollostack.com/#slack)

Easily use the Apollo Client 2 with Reason

## Install and setup

#### yarn
```
yarn add reason-apollo
```

#### bsconfig
Add `reason-apollo` to your `bs-dependencies`:
**bsconfig.json**
```
"bs-dependencies": [
  "reason-react",
  "reason-apollo"
]
```


## Usage 
 
 [here](https://github.com/Gregoirevda/reason-apollo-test-usage) is a repository showing the usage of the package.
 
 
 ### Create the Apollo Client
 
 **Apollo.re**
 ```reason
 module InMemoryCache =
  ApolloInMemoryCache.CreateInMemoryCache(
    {
      type dataObject;
      let inMemoryCacheObject = Js.Nullable.undefined;
    }
  );

/* Create an HTTP Link */
module HttpLink =
  ApolloLinks.CreateHttpLink(
    {
      let uri = "http://localhost:3010/graphql";
    }
  );

module Client =
  ReasonApollo.CreateClient(
    {
      let apolloClient =
        ReasonApollo.createApolloClient(
          ~cache=InMemoryCache.cache,
          ~link=from([|AuthLink.link, ErrorLink.link, HttpLink.link|]),
          ()
        );
    }
  );

 ```
  
  ## Query
  
  ### Query Configuration
  **QueryConfig.re**
  ```reason
  /* Create a GraphQL Query by using the graphql_ppx */ 
  module HeroQuery = [%graphql {|
    query getPokemon($name: String!){
        pokemon(name: $name) {
            name
        }
    }
  |}]; 
  ```

  
  #### Executing the Query
  **YourQuery.re**
  ```reason
  let Query = Client.Instance.Query;

  let make = (_children) => {
  /* ... */
  render: (_) =>
    <Query query=(() => HeroQuery.make(~name="Pikachu", ()))>
      (response => {
        switch response {
           | Loading => <div> (Utils.ste("Loading")) </div>
           | Failed(error) => <div> (Utils.ste(error)) </div>
           | Loaded(result) => <div> (Utils.ste(result##user##name)) </div>
      }})
    </Query>
  }
  ```

  ## Mutation
  
  ### Mutation Configuration
  
  **MutationConfig.re**
  ```reason
  /* Create a mutation with the `graphql-tag` */
  
  let mutation = [@bs] gql({|
    mutation deleteTodo($id: ID!) {
        deleteTodo(id: $id) {
          id
          name
        }
      }
  |});  
  
  /* Describe the result type */
  type todo = {. "name": string, "id": string};
  type data = {. "deleteTodo": todo};
  type response = data;
    
  /* Optional variables passed to the mutation */
    type variables = {. "id": string}; /* or `type variables;` if none */
  ```

  
  ### Executing the Mutation
  **YourMutation.re**
  ```reason
  module DeleteTodo = Apollo.Client.Mutation(MutationConfig);
  
  let variables = {
    "id": "uuid-1"
  };
  
  let make = (_children) => {
  /* ... */
  render: (_) =>
    <DeleteTodo>
      ((
        deleteTodo /* Mutation to call */, 
        result /* Result of your mutation */
      ) => {
          let mutationResponse = switch result {
            | NotCalled => <div>  (Utils.ste("Not Called")) </div>
            | Loading => <div> (Utils.ste("Loading")) </div>
            | Loaded(response) => <div> (Utils.ste(response##deleteTodo##name ++ " deleted")) </div>
            | Failed(error) => <div> (Utils.ste(error)) </div>
          };
        <div>
          <button onClick=((_mouseEvent) => deleteTodo(~variables, ()))> 
            (Utils.ste("Delete Todo")) 
          </button>
          <div> (mutationResponse) </div>
        </div>
      })
    </DeleteTodo>
  }
  ```
