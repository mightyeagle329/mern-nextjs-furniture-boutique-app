# MY NOTES WHEN BUILDING THIS WEB APP

### PROJECT SETUP
- Download the starter project directory from https://github.com/reedbarger/react-reserve
- Install and update the project dependencies by running:
  - `npm install`
  - then `npm update`
- To start up Next.js server and the project, run: `npm run dev`


### WORKING WITH REACT + NEXT.JS
**1. Create App Layout Component, Build Header Component**
- The Layout component is wrapped around the `<Component />` in _app.js file. This will override the default App.js and the layout defined in the Layout component will persist on every page
- Include the Semantic-UI-React stylesheet in Layout.js file
- The Layout component renders the HeadContent, Header Components, and children components
- All the contents inside the Layout component will be rendered on every page. For example, the Header component which contains the navbar will be rendered on every page
- The Header component contains the navbar menu with links that takes user to that particular page
  - Use Link component provided by Next.js to create the links
    - `import Link from 'next/link';`
  - Write a condition that check whether the user is currently logged in or not
  - If user not logged in, show the Login and Signup links and hide the Create link
  - If user is logged in, show the Account link and Logout button 
  - Use Semantic UI to style the Header component

**2. Get Route Data From useRouter Hook, Create Active Links**
- Since we're using a function component, Next provides a React useRouter() hook that gives us information from our router
- `import { useRouter } from 'next/router'`
- When execute, it will return a router object: `const router = useRouter()`
- We can get information about what path we're on from the router object by using the property `router.pathname`
- We can use this information to set the navbar menu item to be active when a user is on that particular route
- In Header.js file:
  - Write an isActive function that checks whether the given route matches with `router.pathname`. Return true or false
  - If it is matched, apply the `active` property to that navbar menu item
  ```js
  import { useRouter } from 'next/router';

  function Header() {
    // When execute, userRouter hook returns a router object
    const router = useRouter();
    const user = false;

    // Check if the given route matches with router.pathname
    // router.pathname gives information about the route the user is on
    function isActive(route) {
      // return true or false
      return route === router.pathname;
    }

    // Apply active style to menu item if the function returns true
    <Menu.Item header active={isActive('/')}>
  }
  ```

**3. Visualize Route Changes with Progress Bar**
- It's always good to display to the users visually what's taking place within the application
- We're going to display a progress bar when loading a new page or when fetching data
- Install nprogress package: `npm i nprogress`
- Next.js provides us with a `Router` object from `next/router`. In this object, we have access to when the route changes. We can write a function that starts the progress bar when the route changes, end the progress bar when route change is complete or encounter error
- In Header.js file:
  - Import the Router object from `next/router`
  - Import nProgress
  - Outside and above the Header component:
    - write a function that starts the progress bar when route change starts
    - Write a function that ends the progress bar when route change ends
    - write a function that ends the progress bar when an error occurs during route change
  ```js
  import Router, { useRouter } from 'next/router';
  import nProgress from 'nprogress';

  Router.onRouteChangeStart = () => nProgress.start();
  Router.onRouteChangeComplete = () => nProgress.done();
  Router.onRouteChangeError = () => nProgress.done();
  ```
- Style the progress bar in static/nprogress.css file and include the stylesheet in Layout.js file


### CREATING API WITH NODE + NEXT SERVER
**1. Node + Next Server with API Routes**
- In our home page in pages/index.js file, when the page loads (when the Home component mounts), we want to fetch the products with API and display them on the page
- Whenever our application is interacting with the outside world, such as fetching data from the database, we can use React's useEffect() hook. Inside this hook, we can call a function that makes an API request
- We will use axios, a tool to help make API requests
- APIs, in the purest sense, are routes. And routes are simple functions. So in order to create routes, we create basic functions
- In Next.js v.9 and newer, Next introduces API Routes. Any file (regular JS file) that is inside the `pages` directory Next.js will create a route for it. So we can create an api folder inside the `pages` directory that contains the route-name JS files and Next.js will automatically create the api routes for those files
- For example, if we want to make a get request to '/api/products' endpoint, we can just create a products.js file inside the pages/api folder. Then in this products.js file, we can write a function that sends a response with the data back to the client
- Another thing to note is that the port that the API request runs on is the same port that the client makes the request. By running on the same port, we can avoid CORS (cross-origin resource sharing) errors
- In pages/index.js file:
  - It's important that functions and components created inside pages directory is export default
  ```js
  import { Fragment, useEffect } from 'react';
  import axios from 'axios';

  function Home() {
    // useEffect hook accepts 2 arguments
    // 1st arg is the effect function
    // Run code inside this function to interact w / outside world
    // 2nd arg is dependencies array
    useEffect(() => {
      getProducts();
    }, []);

    // axios.get() method returns a promise
    // so make the getProduct function an async function
    // the response we get back is in response.data object
    async function getProducts() {
      const url = 'http://localhost:3000/api/products';
      const res = await axios.get(url);
      const { data } = res;
    }

    return <Fragment>home</Fragment>;
  }

  export default Home;
  ```
- In pages/api/products.js file:
  - It's important that functions and components created inside pages directory is export default
  - On every request made, we have access to the request(req) and response(res) objects
  ```js
  import products from '../../static/products.json';

  export default (req, res) => {
    res.status(200).json(products);
  };
  ```

**2. Fetching Data on the Server with getInitialProps**
- With client-side rendering, we would have to wait for the component to mount before we can fetch data. With Next.js, we can fetch data before the component mounts
- We do this using Next's `getInitialProps` function
  - This is an async function
  - This function fetches data on a server
  - Returns with the response data as an object
  - We can pass this object as props to our component
  - Also note that this object props will be merged with existing props
- Now, in order to pass the response data object coming from an API request as props to a component, we need to setup the `<Component />` in our custom _app.js file to receive pageProps, data object made available as props prior to the component mounts 
- In pages/_app.js file:
  ```js
  import App from 'next/app';
  import Layout from '../components/_App/Layout';

  class MyApp extends App {
    static async getInitialProps({ Component, ctx }) {
      let pageProps = {};

      // first check to see if there exists an initial props of a given component
      // if there is, execute the function that accepts context object as an argument
      // this is an async operation
      // assign the result to pageProps object
      if (Component.getInitialProps) {
        pageProps = await Component.getInitialProps(ctx);
      }

      return { pageProps };
    }

    // destructure pageProps objects that's returned from getInitialProps funct
    // the <Component /> is the component of each page
    // each page component now has access to the pageProps object
    render() {
      const { Component, pageProps } = this.props;
      return (
        <Layout>
          <Component {...pageProps} />
        </Layout>
      );
    }
  }

  export default MyApp;
  ```
- In pages/index.js file:
  ```js
  import React, { Fragment, useEffect } from 'react';
  import axios from 'axios';

  function Home({ products }) {
    console.log(products);

    return <Fragment>home</Fragment>;
  }

  // Fetch data and return response data as props object
  // This props object can be passed to a component prior to the component mounts
  // It's an async function
  // NOTE: getServerSideProps does the same thing as getInitialProps function
  export async function getServerSideProps() {
    // fetch data on server
    const url = 'http://localhost:3000/api/products';
    const response = await axios.get(url);
    // return response data as an object
    // note: this object will be merged with existing props
    return { props: { products: response.data } };
  };

  export default Home;
  ```
- **The getInitialProps method:**
  - Docs: https://nextjs.org/docs/api-reference/data-fetching/getInitialProps
  - `getInitialProps` enables server-side rendering in a page and allows you to do initial data population, it means sending the page with the data already populated from the server. This is especially useful for SEO
  - NOTE: `getInitialProps` is deprecated. If using Next.js 9.3 or newer, it's recommended to use `getStaticProps` or `getServerSideProps` instead of `getInitialProps`
  - These new data fetching methods allow you to have a granular choice between static generation and server-side rendering
  - Static generation vs. server-side rendering: https://nextjs.org/docs/basic-features/pages
- **Two forms of pre-rendering for Next.js:**
  - **Static Generation (Recommended):** The HTML is generated at **build time** and will be reused on each request. To make a page use Static Generation, either export the page component, or export `getStaticProps` (and `getStaticPaths` if necessary). It's great for pages that can be pre-rendered ahead of a user's request. You can also use it with Client-side Rendering to bring in additional data
  - **Server-side Rendering:** The HTML is generated on **each request**. To make a page use Server-side Rendering, export `getServerSideProps`. Because Server-side Rendering results in slower performance than Static Generation, use this only if absolutely necessary


### USING MONGODB WITH ATLAS
**1. Configure Mongo Atlas, Connect to Database**
- MongoDB Atlas: https://www.mongodb.com/cloud
- Mongo Atlas is a cloud database service that can host our MongoDB database on a remote server
- Once signed in to MongoDB Cloud, create a new project and choose the free tier. Name it FurnitureBoutique
- Then create a new cluster and give the cluster a name: FurnitureBoutique
- **Connecting the database to our application:**
  - First, we want to whitelist our connection IP address
    - From the project cluster dashboard, click on the Connect button
    - We want to allow our IP address be accessed anywhere. This will prevent potential errors in the future when we deploy our app in production. Problems like database denying access to our application due to the IP address we're trying to connect from
    - To do so, click on the Network Access on the left menu under Security. Then click the Allow Access From Anywhere button
  - Second, create a root database user
    - Create a user name and password
  - Third step is choose a connection method
    - Select the Connect your application option
    - Then what we want is the srv string. Copy the path to the clipboard
    - We just need to replace the username and password that we created for the root user earlier
- **Connect to database:**
  - In next.config.js file:
    - All of our environment variables are stored in this file
    - `MONGO_SRV: "<insert mongodb-srv path here>"`
    - Replace the password and dbname
    - Must restart the server
  - In utils/connectDb.js file:
    - In order to connect to the database we're going to use Mongoose package
    - We use Mongoose quite a lot when working with database
    - Install Mongoose: `npm i mongoose`
    - Write a connectDB function that connects our application to the database
    ```js
    import mongoose from 'mongoose';

    const connection = {};

    // Connect to database
    async function connectDB() {
      // If there is already a connection to db, just return
      // No need to make a new connection
      if (connection.isConnected) {
        // Use existing database connection
        console.log('Using existing connection');
        return;
      }
      // Use new database connection when connecting for 1st time
      // 1st arg is the mongo-srv path that mongo generated for our db cluster
      // The 2nd arg is options object. Theses are deprecation warnings
      // mongoose.connect() returns a promise
      // What we get back from this is a reference to our database
      const db = await mongoose.connect(process.env.MONGO_SRV, {
        useCreateIndex: true,
        useFindAndModify: false,
        useNewUrlParser: true,
        useUnifiedTopology: true
      });
      console.log('DB Connected');
      connection.isConnected = db.connections[0].readyState;
    }

    export default connectDB;
    ```
- **Call the connectDB function in routes:**
  - Finally, import and execute the connectDB function in the request routes files which are in the pages/api folder
  - Must restart the server
  - For example, import and execute the function in pages/api/products.js file
    - Execute the connectDB function at the very top and outside of the request route function
    ```js
    import connectDB from '../../utils/connectDb';

    // Execute the connectDB function to connect to MongoDB
    connectDB();
    ```
- If we're successfully connected to MongoDB, we should be able to see "DB Connected" logged in the console
- Lastly, add the next.config.js file to the .gitignore file

**2. Create Products Collection, Model Product Data**
- **Create products collection MongoDB by importing our static products data:**
  - On MongoBD project dashboard page, click on the Command Line Tools menu item at the top
  - In the Data Import and Export Tools section, copy the script for 'mongoimport'
  - In the terminal at the root of the project, paste in the script and specify the collection information
  - In our example, we want to import our static products json data into MongoDB Atlas
  - We'll call our collection products, type is json, provide the path to the data file, and add the --jsonArray flag
  - Use npx before the script
  - `npx mongoimport --uri mongodb+srv://<USERNAME>:<PASSWORD>@furnitureboutique.pikdk.mongodb.net/<DATABASE> --collection products --type json --file ./static/products.json --jsonArray`
  - If successful, we'll be able to see our products collection in MongoDB
- **Model Product data:**
  - We use Mongoose package to connect our application to the database. Now we will use Mongoose as ORM (Object Relational Mapper). It's a tool that's going to specify what each document must have for it to be added to a collection. What it must have in terms of properties and the corresponding data types and other conditions
  - An alternative word for a model is a schema
  - To create a Product model, we first need to define the product schema which contains the required fields and then call `mongoose.model()` to create a new model based on the schema we define
  - In models/Product.js file:
    - Use mongoose to create a new schema by using `new mongoose.Schema()`
    - This method takes an object as an argument and on this object we can specify all of the fields that a given document must have
    - The return result from this method we'll save to a variable ProductSchema
    - We'll use the npm package shortid to generate unique ids
    - Install shortid: `npm i shortid`
    - Generate a unique id by calling `shortid.generate()`
    - Call mongoose.model() method to create a new model
      - 1st arg is the name of the model
      - 2nd arg is the schema
    - We also want to check if the Product model already exists in our connected database. If it does, use the existing model. If it doesn't, then create the Product model
    ```js
    import mongoose from 'mongoose';
    import shortid from 'shortid';

    const { String, Number } = mongoose.Schema.Types;

    const ProductSchema = new mongoose.Schema({
      name: {
        type: String,
        required: true
      },
      price: {
        type: Number,
        required: true
      },
      sku: {
        type: String,
        unique: true,
        default: shortid.generate()
      },
      description: {
        type: String,
        required: true
      },
      mediaUrl: {
        type: String,
        required: true
      }
    });

    export default mongoose.models.Product ||
      mongoose.model('Product', ProductSchema);
    ```
- **Fetch Products From Mongo Database:**
  - In pages/api/products.js file:
    - Import the Product model and call the find() method on Product to retrieve the products from db
    - This is an async operation, so make the route function an async function
    ```js
    import Product from '../../models/Product';
    import connectDB from '../../utils/connectDb';

    // Execute the connectDB function to connect to MongoDB
    connectDB();

    export default async (req, res) => {
      const products = await Product.find();
      res.status(200).json(products);
    };
    ```
- Now when we make a request to `/api/products` endpoint, we should get back the products array coming from the MongoDB database

**3. Build Product Cards, Make Components Responsive**
- Semantic UI Card: https://react.semantic-ui.com/views/card/
- We'll use Semantic UI Card component to style the products on our home page
- The pages/index.js file renders the ProductList.js component
  - Pass the products array as props to ProductList component
  ```js
  import ProductList from '../components/Index/ProductList';

  function Home({ products }) {
    // console.log(products);
    return <ProductList products={products} />;
  }
  ```
- In components/Index/ProductList.js file
  - Destructure the products props
  - Write a mapProductsToItems function that maps over the products array and returns a new array of product objects
    - This product object defines keys and values that we can use to render the product in the Semantic UI Card component
  - In the Card component, specify the number of items per row
  - Add the stackable attribute to the `<Card.Group />` component so the items will stack on top of each other on smaller size screens
  - Do the same thing for the navbar menu items in Header.js component: `<Menu fluid inverted id='menu' stackable>`
  ```js
  import { Card } from 'semantic-ui-react';

  function ProductList({ products }) {
    function mapProductsToItems(products) {
      return products.map((product) => ({
        header: product.name,
        image: product.mediaUrl,
        meta: `$${product.price}`,
        color: 'teal',
        fluid: true,
        childKey: product._id,
        href: `/product?_id=${product._id}`
      }));
    }

    return (
      <Card.Group
        stackable
        itemsPerRow='3'
        centered
        items={mapProductsToItems(products)}
      />
    );
  }

  export default ProductList;
  ```


### FETCHING APP DATA FROM API
**1. Get Product By Id**
- When we click on a product, we want to direct user to the product detail page. We make an API request to get the product by its id
- In Next.js, we're able to fetch data before the component mounts. So we can make use of Next's getInitialProps function to fetch the data
- In pages/product.js file:
  - getInitialProps function automatically receives the context object as an argument
  - One of the properties in context object is query. We can use query string to get the product id to make the request
  - This function returns the response data object which we can pass to our Product component as props
  ```js
  import axios from 'axios';

  function Product({ product }) {
    console.log(product);
    return <p>product</p>;
  }

  Product.getInitialProps = async ({ query: { _id } }) => {
    const url = 'http://localhost:3000/api/product';
    const payload = { params: { _id } };
    const response = await axios.get(url, payload);
    return { product: response.data };
  };

  export default Product;
  ```
- Now let's create the API endpoint/route for the endpoint we defined in pages/product.js page
- In pages/api/product.js file:
  - Import the Product model and call the findOne() method on Product
  - The findOne() method is like a filter method. We want to filter by the _id property
  ```js
  import Product from '../../models/Product';

  export default async (req, res) => {
    // req.query is an object
    const { _id } = req.query;
    const product = await Product.findOne({ _id });
    res.status(200).json(product);
  };
  ```

**2. Style Product Detail Page**
- We will use three components to create and style the product detail page
- In components/Product folder:
  - ProductSummary.js - renders the AddProductToCart.js component
  - ProductAttributes.js
  - AddProductToCart.js
- In pages/product.js file:
  - The product route page renders the ProductSummary.js and ProductAttributes components
  - Each component receives the product object as props
  ```js
  // Spreading the product object as props using the object spread operator
  <Fragment>
    <ProductSummary {...product} />
    <ProductAttributes {...product} />
  </Fragment>
  ```
- In components/Product/ProductSummary.js file:
  - Destructure only the keys needed from the product object props
  ```js
  import { Item, Label } from 'semantic-ui-react';
  import AddProductToCart from './AddProductToCart';

  function ProductSummary({ _id, name, mediaUrl, sku, price }) {
    return (
      <Item.Group>
        <Item>
          <Item.Image size='medium' src={mediaUrl} />
          <Item.Content>
            <Item.Header>{name}</Item.Header>
            <Item.Description>
              <p>${price}</p>
              <Label>SKU: {sku}</Label>
            </Item.Description>
            <Item.Extra>
              <AddProductToCart productId={_id} />
            </Item.Extra>
          </Item.Content>
        </Item>
      </Item.Group>
    );
  }

  export default ProductSummary;
  ```
- In components/Product/ProductAttributes.js file:
  - Destructure only the keys needed from the product object props
  ```js
  import { Button, Header } from 'semantic-ui-react';

  function ProductAttributes({ description }) {
    return (
      <>
        <Header as='h3'>About this product</Header>
        <p>{description}</p>
        <Button
          icon='trash alternate outline'
          color='red'
          content='Delete Product'
        />
      </>
    );
  }

  export default ProductAttributes;
  ```
- In components/Product/AddProductToCart.js file:
  ```js
  import { Input } from 'semantic-ui-react';

  function AddProductToCart(productId) {
    return (
      <Input
        type='number'
        min='1'
        placeholder='Quantity'
        value={1}
        action={{
          color: 'orange',
          content: 'Add to Cart',
          icon: 'plus cart'
        }}
      />
    );
  }

  export default AddProductToCart;
  ```

**3. Base URL Helper**
- When we fetch data in a development environment, we make request to `http:localhost:3000` on local server. And when we're in production, we're going to use the deployment URL
- Let's write a base URL helper function that detects whether we're in a production or development environment. We can dynamically determine this whether that's the case or not with the help of a environment variable
- In utils/baseUrl.js file:
  ```js
  const baseUrl =
    process.env.NODE_ENV === 'production'
      ? 'https://deployment-url.now.sh'
      : 'http://localhost:3000';

  export default baseUrl;
  ```
- Then wherever we use a URL to fetch data, we can replace the base URL with our baseUrl helper to generate the base URL dynamically
- In pages/index.js and pages/product.js files:
  - Import the baseUrl helper function
  - For the url variable, use template literal and interpolate the baseUrl variable
  ```js
  import baseUrl from '../utils/baseUrl';

  const url = `${baseUrl}/api/products`;
  ```


### ADDING CRUD FUNCTIONALITY, UPLOADING IMAGE FILES
**1. Delete A Product Functionality**
- When a user clicks on the Delete Product button, we want to display a modal asking the user to confirm the product deletion
- The modal contains the cancel button and Delete button
- To implement the modal functionality, we want to create a state for the modal to keep track of modal state in our application. We can use React useState() hook
- When the Cancel button is clicked, we just want to close the modal, setting the modal state to false
- When the Delete button is clicked, we want to make a delete API request to backend and delete the product based on id. Then redirect user to products index page
- In components/Product/ProductAttributes.js file:
  - Use Semantic UI Modal component to make the modal
  - Use React useState() to create the modal state. Default value is set to false
    - When the Delete Product button is clicked, set modal state to true
    - When Cancel button is clicked, set modal state to false. This will close the modal
  - Use useRouter hook from Next to redirect
  - Write a handleDelete function that makes a delete API request using axios to delete the product based on id
  ```js
  import React, { Fragment, useState } from 'react';
  import { useRouter } from 'next/router';
  import { Button, Header, Modal } from 'semantic-ui-react';
  import axios from 'axios';
  import baseUrl from '../../utils/baseUrl';

  function ProductAttributes({ description, _id }) {
    const [modal, setModal] = useState(false);
    const router = useRouter();

    async function handleDelete() {
      const url = `${baseUrl}/api/product`;
      const payload = { params: { _id } };
      await axios.delete(url, payload);
      // redirect to home page after delete product
      router.push('/');
    }

    return (
      <Fragment>
        <Header as='h3'>About this product</Header>
        <p>{description}</p>
        <Button
          icon='trash alternate outline'
          color='red'
          content='Delete Product'
          onClick={() => setModal(true)}
        />
        <Modal open={modal} dimmer='blurring'>
          <Modal.Header>Confirm Delete</Modal.Header>
          <Modal.Content>
            <p>Are you sure you want to delete this product?</p>
          </Modal.Content>
          <Modal.Actions>
            <Button content='Cancel' onClick={() => setModal(false)} />
            <Button
              negative
              icon='trash'
              labelPosition='right'
              content='Delete'
              onClick={handleDelete}
            />
          </Modal.Actions>
        </Modal>
      </Fragment>
    );
  }

  export default ProductAttributes;
  ```
- In pages/api/product.js file:
  - In the api routes for product, we want to be able to handle different types of requests such as create, read, update, and delete
  - For each request, we have access to the request and response objects. Using `req.method`, we can figure out what type of request it is
  - And based on the type of request, we can write the appropriate type of route handler to handle the request
  - We can use the switch statement to handle different types of requests
  - For now, we have a get and delete requests of a product
  ```js
  import Product from '../../models/Product';

  export default async (req, res) => {
    switch (req.method) {
      case 'GET':
        await handleGetRequest(req, res);
        break;
      case 'DELETE':
        await handleDeleteRequest(req, res);
        break;
      default:
        res.status(405).send(`Method ${req.method} not allowed`); //405 means error with request
        break;
    }
  };

  async function handleGetRequest(req, res) {
    const { _id } = req.query;
    const product = await Product.findOne({ _id });
    res.status(200).json(product);
  }

  async function handleDeleteRequest(req, res) {
    const { _id } = req.query;
    await Product.findOneAndDelete({ _id });
    // status code 204 means success and no content is sent back
    res.status(204).json({});
  }
  ```













  


## RESOURCES
- Next.js docs: https://nextjs.org/docs/getting-started
- Semantic UI docs: https://react.semantic-ui.com/

## NPM PACKAGES USED IN THIS PROJECT
- react, react-dom
- next
- semantic-ui-react
- nprogress
- mongoose