---
layout: post
title:      "Using Axios to make API calls in React"
date:       2021-02-04 02:15:57 -0500
permalink:  using_axios_to_make_api_calls_in_react
---


Many web applications need to make API calls to a backend REST API server.  Axios is a lightweight Promise-based HTTP client for JavaScript that is similar to the native JavaScript Fetch API.

Axios makes it easy to send asynchronous HTTP request to REST endpoints and perform CRUD operations. Axios can by used in a plain JavaScript application

## Add Axios to your React project

Create a new app:

npx create-react-app my-online-store

Move inside our new project:

cd my-online-store

Install dependencies.
```npm install axios
```

Start the project.
```
npm start
```

## Fetch data with GET request 
### Data fetching in a class component
First, you import React and Axios so you can use them in the class component. 
Then you make the GET request in the componentDidMount lifecycle method.

```
import React from 'react';
import axios from 'axios';

export default class ProductContainer extends React.Component {
  state = { products: [] };

  componentDidMount() {
    axios.get(`https://gorest.co.in/public-api/products`)
      .then(resp => {
        this.setState({ products: resp.data }))
      })
			.catch(error => console.log(error));
  }

  render() {
    return (
      <ul>
        { this.state.products.map(product => <li>{ product.name }</li>)}
      </ul>
    )
  }
}
```
If you use the JavaScript native fetch API of the browser, there is a two-step process when handing JSON data. The first is to make the actual request, and then the second is to call the .json() method on the response.

When we use axios.get(url) to get a promise that returns a response object, we do not need the second step of passing the results of the http request to the .json() method. Axios performs automatic JSON data transformation.


### Data Fetching in a functional component with React Hooks
First, you import React, useState, and useEffect in the component. 
Then you make the GET request inside the useEffect hook.
```
import React, { useState, useEffect } from 'react';
import axios from 'axios';
 
function ProductList() {
  const [data, setData] = useState({ products: [] });
 
  useEffect(
	    axios.get(`https://gorest.co.in/public-api/products`)
      .then(resp => {
          setData(resp.data))
      })
			.catch(error => console.log(error));
    );

  }, []);
 
  return (
    <ul>
      { data.products.map(product => (
        <li key={product.id}> { product.name } </li>
      ))}
    </ul>
  );
}
```

## Make a Post Request 
Using Axios, we can create POST requests using the .post() method and passing the data as a second parameter. 

```
import React from 'react';
import axios from 'axios';

export default class AddProduct extends React.Component {
  state = {
    name: '""
  }

  handleChange = event => {
    this.setState({ name: event.target.value });
  }

  handleSubmit = event => {
    event.preventDefault();

    const product = {
        name: this.state.name
    };
		
		const token = 'my secret token'
		
		const config = {
        headers: { Authorization: `Bearer ${token}` }
    };
		
    axios.post(`https://gorest.co.in/public-api/products`, { product }, config)
      .then(resp => {
        console.log(resp);
        console.log(resp.status);
      })
  }

  render() {
    return (
      <div>
        <form onSubmit={this.handleSubmit}>
          <label>
            Person Name:
            <input type="text" name="name" onChange={this.handleChange} />
          </label>
          <button type="submit">Add</button>
        </form>
      </div>
    )
  }
}
```

## Make a Delete Request
```
import React from 'react';
import axios from 'axios';

export default class DeleteProduct extends React.Component {
  state = {
    id: '',
  }

  handleChange = event => {
    this.setState({ id: event.target.value });
  }

  handleSubmit = event => {
    event.preventDefault();

    axios.delete(`https://gorest.co.in/public-api/products/${this.state.id}`)
      .then(res => {
        console.log(res);
        console.log(res.data);
      })
  }

  render() {
    return (
      <div>
        <form onSubmit={this.handleSubmit}>
          <label>
            Person ID:
            <input type="text" name="id" onChange={this.handleChange} />
          </label>
          <button type="submit">Delete</button>
        </form>
      </div>
    )
  }
}
```
## Advantage of Axios over Fetch API
* Performs automatic JSON data transformation
* Supports upload progress
* Has a way to set a response timeout
* Axios throws an error when a request fails
*  Set default values for common headers (like Authorization) on outgoing requests. This can be useful if you are authenticating to a server on every request.
*  Create reusable instances with baseUrl, headers, and other configuration already set up.
