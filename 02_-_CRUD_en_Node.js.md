# CRUD en Node.js

## Insertar

### Vista de crear un producto

Agrega un link a una vista nueva que muestra un formulario de productos, recuerda que el proceso es:

1. Crear un controlador.

2. Crear la función dentro del controlador.

3. Crear la ruta dentro de nuestro archivo de rutas.

4. En la función del controlador crear la vista que vamos a regresar.

La información que vamos a capturar es nombre y precio, nuestro archivo tendrá un código similar a:

```handlebars
<!-- views/products/create.hbs -->
<h1>Add a new product</h1>
<form method="POST" action="/products">
    <div>
        <label for="name">Name: </label>
        <input type="text" id="name" name="name">
    </div>
    <div>
        <label for="price">Price: </label>
        <input type="text" id="price" name="price">
    </div>
    <div>
        <input type="submit" value="Store">
    </div>
</form>
```

Nota que lo estamos enviando a la ruta `/products` a través del método `POST`. Por lo tanto en nuestras rutas debemos de manejar esta petición. 

```js
// routes/app.js

// ...

// Identifica la ruta /products/create y la envía al controlador
router.get('/products/create', ProductsController.create);
// Almacena el producto
router.post('/products', ProductsController.store);

// ...
```

### Leer el input del usuario

Para leer el input del usuario primero debemos de importar el módulo de [body-parser](https://www.npmjs.com/package/body-parser) (incluído en express.js).

```js
// index.js

// Importa el paquete de express
let express = require('express');
// Obtiene una instancia de express
let app = express();

// Importa el modulo para leer el input del usuario
let bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({ extended: true }));

// ...
```

Después en nuestra función de almacenamiento en nuestro `ProductsController.js` podemos imprimir el contenido del `input` del usuario usando `console.log(req.body)`.

```js
// controllers/ProductsController.js
// Reglas para la respuesta para la petición "/products/create"
exports.create = (req, res) => {
  res.render('products/create');
}

// Almacena el producto
exports.store = (req, res) => {
  console.log(req.body);
  res.send('Producto almacenado');
}
```

```bash
node index.js
# App listening on port 3000! (http://localhost:3000)
# { name: 'iPad XI', price: '2500' }
```

### Almacenar el producto

Ya que tenemos la información del usuario en nuestro `req.body` podemos almacenar el producto en nuestra base de datos.

El primer paso será modificar nuestro modelo de Producto para agregar la función `create`, en la cual enviaremos los datos del producto y esta función se encargará de almacenarlo en la base de datos.

```js
// models/Products.js
// ...
// Almacen en la base de datos el producto
exports.create = (product) => {
  return knex('products')
    .insert({
      name: product.name,
      price: product.price,
      description: product.description
    });
}

```

Después de crear la función de almacenamiento en el modelo, modificaremos la lógica de nuestro controlador para que haga el `parsing` correcto del input del usuario y almacene el producto.

```js
// controllers/ProductsController.js
// Importa el modelo de productos
let ProductModel = require('../models/Product')
// ...
// Almacena el producto
exports.store = (req, res) => {
  // Crea un objeto con la información del usuario
  let product = {
    name: req.body.name,
    price: req.body.price,
    description: req.body.description
  };
  // Crea el producto
  ProductModel.create(product)
    .then((id) => {
      // Al finalizar la creación, reenvía al usuario a la página
      // inicial
      res.redirect('/');
    });
}

```

## Mostrar

Al igual que para insertar un producto, para mostrar el producto debemos de:

1. Crear una función dentro del controlador para que muestre el producto.

2. Crear la ruta a la función de nuestro controlador.

3. En la función del controlador crear la vista que vamos a regresar con la información del producto.

### Mostrar un producto

Para mostrar un producto, primero debemos de modificar nuestro modelo de `models/Product.js` para que pueda obtener la información del producto de nuestra base de datos.

Esto lo puedes lograr con la función `select` de Knex.

```js
// models/Product.js

// ...

// Obtiene la información de un producto por su id
exports.find = (id) => {
  return knex
    .select('*')
    .from('products')
    .where('id', id)
    .first();
}
```

Ahora que tenemos la función `find` dentro de nuestro modelo, debemos de modificar nuestro controlador para que muestre el producto indicado.

```js
// controllers/ProductsController.js

// ...

// Muestra el producto
exports.show = (req, res) => {
  // Obtiene el id que viene en la url
  let id = req.params.id;
  // Busca dentro de la base de datos el producto con el id indicado
  ProductModel.find(id).then((product) => {
    // Si el producto no existe entonces
    if (product == null) {
      // Regresa el error 404
      res.status(404).send('Not found');
      return;
    }
    // Si el producto existe entonces muestra la vista products/show.hbs
    // con la información del producto
    res.render('products/show', {product: product});
  });
}
```

En la vista `views/products/show.hbs`  utilizamos la información del producto:

```handlebars
<h1>Product</h1>
<table>
    <tr>
        <th>Name</th>
        <td>{{product.name}}</td>
    </tr>
    <tr>
        <th>Price</th>
        <td>{{product.price}}</td>
    </tr>
</table>
```

Finalment modificamos nuestro archivo de rutas para que incluya la ruta para mostrar el producto.

```js
// routes/app.js

// router.get('/products/create', ProductsController.create);

// Implementa ver el detalle de un producto
router.get('/products/:id', ProductsController.show);


// router.post('/products', ProductsController.store);
```


