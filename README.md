# Trabajo final IT Backend Bootcamp
Wave19 - Grupo 05 - Martin Riani Tonna

![technology Java](https://img.shields.io/badge/technology-Java-blue.svg)
![technology Maven](https://img.shields.io/badge/technology-Maven-blue.svg)
![framework](https://img.shields.io/badge/framework-springboot:2.7.2.RELEASE-green)

- [1. Levantar el servicio](#1-Levantar-el-servicio)
- [2. Base de datos y datos](#2-Base-de-datos-y-datos)
    - [2.1. H2](#21-H2)
    - [2.2. MySQL](#22-MySQL)
- [3. Interactuar con el servicio ](#3-Interactuar-con-el-servicio)
    - [3.1. US01. Ingresar lote en warehouse de fulfillment](##3.1.-US01.-Ingresar-lote-en-warehouse-de-fulfillment)
    - [3.2. US02. Registrar Venta: Agregar producto al carrito de compras](#3.2.-US02.-Registrar-Venta:-Agregar-producto-al-carrito-de-compras)
    - [3.3. US03. Consultar ubicación de un producto en el warehouse](#3.3.-US03.-Consultar-ubicación-de-un-producto-en-el-warehouse)
    - [3.4. US04. Consultar el stock de un producto en todos los warehouses](#3.4.-US04.-Consultar-el-stock-de-un-producto-en-todos-los-warehouses)
    - [3.5. US05. Consultar fecha de vencimiento por lote](#3.5.-US05.-Consultar-fecha-de-vencimiento-por-lote)
    - [3.6. US06. Gestionar el alta, baja, modificación y eliminación de un producto](#3.6.-US06.-Gestionar-el-alta,-baja,-modificación-y-eliminación-de-un-producto)

- [4. UML](#4-UML)
- [5. DER](#5-DER)

## 1 Levantar el servicio

Para levantar como ambiente local, previamente se debe crear un esquema en la base de datos MySQL con el nombre:
***produtos***.

i. Settear las configuraciones para la ejecución utilizando ***java 17*** y creando las siguientes variables de
entorno, ***SCOPE*** para indicar el ambiente, ***dbuser*** y ***dbpass*** para indicar el usuario y contraseña de la
base de datos local.

`APPLICATION=app; SCOPE=local; dbuser=user; dbpass=pass`

ii. Ejecutar, ya sea en modo Run o en modo Debug

iii. Cuando se encuentre corriendo se podrá visualizar el puerto:

Esto implica que se han generado las tablas en la base de datos local con datos precargados. Estos datos se encuentran
en

    `src/main/resources/data-local.sql`

## 2 Base de datos y datos

### 2.1 H2

Para interactuar con la base directamente si fuera necesario

i. Realizar un login

`GET /api/v1/sign-in?username={user}&password={pass}`

Los usuarios que realizan login pueden ser ***Representante*** o ***Buyer/Seller***. Esta distinción será importante al
interactuar con los endpoints pero para esta instancia cualquiera de los cargados sirve, por ej: ***user_one*** con
pass ***contra123***

ii. la request anterior devuelve un token, debemos copiarlo

![Token.png](docs/img/Token.png)

iii. Para este paso es necesario contar con la extensión ***ModHeader*** para Chrome o alguna similar.

Habilitar la extensión y agregar un header llamado ***Authorization*** cuyo valor es el token obtenido anteriormente:

![ModHeader.png](docs/img/ModHeader.png)

iv. Con la extensión ModHeader configurada, acceder a través del navegador a `http://localhost:8080/h2-console` y luego
hacer click en ***Connect***

**IMAGEN DE REFERENCIA**

![H2Login.png](docs/img/H2Login.png)

**IMAGEN DE REFERENCIA**

![H2DBConsole.png](docs/img/H2DBConsole.png)

### 2.2 MySQL

Es necesario crear el schema `products` en nuestra base de datos local, actualmente esta configurado el archivo `src/main/resources/application-local.yml` con las siguientes propiedades:

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/products
    username: ${dbuser}
    password: ${dbpass}
  jpa:
    show-sql: true
    database-platform: org.hibernate.dialect.MySQL8Dialect
    defer-datasource-initialization: true

    generate-ddl: true
    hibernate:
      ddl-auto: create-drop
  sql:
    init:
      mode: always
      data-locations: classpath:data-local.sql
```

Cargará automaticamente datos de prueba contenidos en : `src/main/resources/data-local.sql`

## 3 Interactuar con el servicio

En esta sección se muestran los
llamados a los diferentes endpoints
desde Postman. En cada caso se indica
si el token debe correspondeer a un usuario
***MANAGER*** , ***ADMIN***, ***CUSTOMER***, ***OPERATOR***, ***SELLER 1*** o ***SELLER 2***.

A continuación un listado de los usuarios disponibles y sus roles:

| Rol      | Username                    | Password  |
|----------|-----------------------------|-----------|
| MANAGER  | user_manager_test           | 123456    | 
| ADMIN    | user_admin_test             | 123456    | 
| CUSTOMER | user_customer_test          | 123456    |
| OPERATOR | user_operator_test          | 123456    | 
| SELLER 1 | user_customer_test_seller_1 | 123456    | 
| SELLER 2 | user_customer_test_seller_2 | 123456    | 


Una vez obtenido el token de la misma manera que se hizo para
configurar el acceso a la consola de H2, se debe agregar el
header en cada request que se haga al servicio, como se ve en la siguiente imagen:

![AuthHeaderPostman.png](docs/img/AuthHeaderPostman.png)

**NOTA**: La colección de postman incluida en
`/docs/SPRINT3.postman_collection.json`
tiene una carpeta llamada LOGIN, en la cual vienen llamadas
a los distintos usuarios con sus roles, y mediante un script en el apartado test
se setean como variable de entorno global en {{token}}, y luego en cada llamada a la app se usa de esta manera:

![TokenPostmanVar.png](docs/img/TokenPostmanVar.png)

### 3.1. US01. Ingresar lote en warehouse de fulfillment

#### 3.1.1 Crear Inbound Order

* **Verbo:** `POST`

* **Url:** `http://localhost:8080/api/v1/fresh-products/inboundorder`

* **Función:** Permite crear una Inbound Order con sus Batches asociados

* **Tipo de usuario requerido:** `OPERATOR`

* **Body de ejemplo:**
```json
{
  "order_number": 2,
  "order_date": "29-01-2023",
  "section": {
    "sectionCode":"1",
    "warehouseCode":"1"
  },
  "batch_stock": [
    {
      "batch_number": 7,
      "productId": 1,
      "currentTemperature": 19.0,
      "minimumTemperature": 16.0,
      "initialQuantity": 5,
      "currentQuantity": 5,
      "manufacturingDate": "14-06-2021",
      "manufacturingTime": "00:00:00",
      "dueDate": "15-07-2021"
    }
  ]
}
```

---

#### 3.1.2 Modificar Inbound Order

* **Verbo:** `PUT`

* **Url:** `http://localhost:8080/api/v1/fresh-products/inboundorder`

* **Función:** Permite modificar una Inbound Order existente, actualizando sus Batches asociados (se indica el batchNumber en el body) o agregandole nuevos

* **Tipo de usuario requerido:** `OPERATOR`

* **Body de ejemplo:**
```json
{
  "order_number": 3,
  "order_date": "29-01-2023",
  "section": {
    "sectionCode":"1", 
    "warehouseCode":"1"
  }, 
  "batch_stock": [
    {
      "batchNumber":3,
      "productId": 1,
      "currentTemperature": 19.0,
      "minimumTemperature": 16.0,
      "initialQuantity": 7,
      "currentQuantity": 7,
      "manufacturingDate": "01-06-2021",
      "manufacturingTime": "00:00:00",
      "dueDate": "10-07-2021"
    },
    {
      "batchNumber":4,
      "productId": 1,
      "currentTemperature": 19.0,
      "minimumTemperature": 16.0,
      "initialQuantity": 20,
      "currentQuantity": 20,
      "manufacturingDate": "01-06-2021",
      "manufacturingTime": "00:00:00",
      "dueDate": "10-07-2021"
    }
  ]
}
```


### 3.2. US02. Registrar Venta: Agregar producto al carrito de compras

#### 3.2.1 Crear Purchase Order

* **Verbo:** `POST`

* **Url:** `http://localhost:8080/api/v1/fresh-products/orders`

* **Función:** Permite crear una Purchase Order

* **Tipo de usuario requerido:** `CUSTOMER`

* **Body de ejemplo:**
```json
{
  "date": "30-01-2023",
  "buyer_id": 1,
  "order_status": {
    "status_code": "carrito"
  },
  "products": [
    {
      "product_id": 1,
      "quantity": 2
    },
    {
      "product_id": 2,
      "quantity": 6
    }
  ]
}
```

---

#### 3.2.2 Modificar Purchase Order

* **Verbo:** `PUT`

* **Url:** `http://localhost:8080/api/v1/fresh-products/orders/{idOrder}`

* **Función:** Permite modificar una Purchase Order, ***reemplazando todos sus datos***

* **Tipo de usuario requerido:** `CUSTOMER`

* **Body de ejemplo:**
```json
{
  "purchase_order": {
    "date": "05-01-2023",
    "buyer_id": 1,
    "order_status": {
      "status_code": "carrito"
    },
    "products": [
      {
        "product_id": 1,
        "quantity": 5
      }
    ]
  }
}
```
---

#### 3.2.3 Consultar Purchase Order

* **Verbo:** `GET`

* **Url:** `http://localhost:8080/api/v1/fresh-products/orders/{idOrder}`

* **Función:** Permite obtener el listado de productos y cantidad de una Purchase Order

* **Tipo de usuario requerido:** `CUSTOMER`

---

#### 3.2.4 Consultar Productos disponibles

* **Verbo:** `GET`

* **Url:** `http://localhost:8080/api/v1/fresh-products/list`

* **Función:** Permite obtener un listado de productos vigentes y disponibles

* **Tipo de usuario requerido:** `CUSTOMER`

---

#### 3.2.5 Consultar Productos disponibles por categoría

* **Verbo:** `GET`

* **Url:** `http://localhost:8080/api/v1/fresh-products/list?category={FS, RF, FF}`

* **Función:** Permite obtener un listado de productos vigentes y disponibles de una categoría

* **Tipo de usuario requerido:** `CUSTOMER`

### 3.3. US03. Consultar ubicación de un producto en el warehouse

* **Función:** Permite obtener todos los batches de determinado producto presentes en el warehouse al que pertenece el representante que hace la consulta

* **Tipo de usuario requerido:** `MANAGER`

#### 3.3.1 Consultar ubicacion del stock

* **Verbo:** `GET`

* **Url:**

`http://localhost:8080/api/v1/fresh-products/{idProduct}/batch/list`

---

#### 3.3.2 Consultar ubicacion del stock ordenado

* **Verbo:** `GET`

* **Url:**

`http://localhost:8080/api/v1/fresh-products/{idProduct}/batch/list?order={L, C, F}`

orderType(L|C|F) es opcional, si no se indica por deafult se aplica L.


### 3.4. US04. Consultar el stock de un producto en todos los warehouses

#### 3.4.1 Consultar stock

* **Verbo:** `GET`

* **Url:** `http://localhost:8080/api/v1/fresh-products/{idProduct}/warehouse/list`

* **Función:** Permite conocer el stock disponible de un producto en todos los warehouses

* **Tipo de usuario requerido:** `MANAGER`

### 3.5. US05. Consultar fecha de vencimiento por lote


#### 3.5.1 Consultar próximos a vencer

* **Función:** Permite conocer la cantidad de productos que se encuentran próximos a vencer

* **Tipo de usuario requerido:** `MANAGER`

* **Verbo:** `GET`

* **Url:**

`http://localhost:8080/api/v1/fresh-products/batch/list/due-date/{cantDays}`

---

#### 3.5.2 Consultar próximos a vencer, que pertenecen a una determinada categoría y ordenado

* **Verbo:** `GET`

* **Url:**

`http://localhost:8080/api/v1/fresh-products/batch/list/due-date/{cantDays}?category={FS, RF, FF}&
order={date_asc, date_desc}
`

Tanto order(date_asc|date_desc) como category(FS|RF|FF) son opcionales.

### 3.6. US06. Gestionar el alta, baja, modificación y eliminación de un producto

#### 3.6.1 Agregar un nuevo producto

* **Verbo:** `POST`

* **Url:**
  `http://localhost:8080/api/v1/products`
* **Función:** Dar de alta un producto donde el único seller y dueño es quien realiza la petición.
  ​
* **Tipo de ROL requerido:** `CUSTOMER`
* **Restricción:** El usuario autenticado debe ser el seller del nuevo producto
* **Request Body de ejemplo:**
```json
{
    "product_code": "ABC1234567",
    "description": "Prueba 1",
    "width": 1.0,
    "height": 1.0,
    "length": 1.0,
    "netweight": 1.0,
    "expiration_rate": 1.0,
    "recommended_freezing_temperature": -10.0,
    "freezing_rate": 1.0,
    "sale_price": 200.0,
    "product_type": {
        "code": "FS"
    },
    "seller": {
        "id": 1
    }
}
```

* **Response Body de ejemplo:**

```json
{
  "id": 3,
  "product_code": "ABC1234567",
  "description": "Prueba 1",
  "width": 1.0,
  "height": 1.0,
  "length": 1.0,
  "expiration_rate": 1.0,
  "recommended_freezing_temperature": -10.0,
  "freezing_rate": 1.0,
  "sale_price": 200.0,
  "product_type": {
    "code": "FS",
    "description": "Fresco"
  },
  "seller": {
    "id": 1,
    "name": "Comercio",
    "locality": "Florida",
    "province": "Santiago del Estero"
  }
}
```

---

#### 3.6.2 Modificar un producto

* **Verbo:** `PUT`

* **Url:**
  `http://localhost:8080/api/v1/products/{idProducto}`
* **Función:** Modificar los datos de un producto existente.
  ​
* **Tipo de ROL requerido:** `CUSTOMER`
* **Restricción:** El usuario autenticado debe ser el seller del producto
* **Request Body de ejemplo:**
```json
{
    "product_code": "ABC1234567",
    "description": "Prueba 1",
    "width": 1.0,
    "height": 1.0,
    "length": 1.0,
    "netweight": 1.0,
    "expiration_rate": 1.0,
    "recommended_freezing_temperature": -10.0,
    "freezing_rate": 1.0,
    "sale_price": 200.0,
    "product_type": {
        "code": "FS"
    },
    "seller": {
        "id": 1
    }
}
```

* **Response Body de ejemplo:**

```json
{
  "id": 3,
  "product_code": "ABC1234567",
  "description": "Prueba 1",
  "width": 1.0,
  "height": 1.0,
  "length": 1.0,
  "expiration_rate": 1.0,
  "recommended_freezing_temperature": -10.0,
  "freezing_rate": 1.0,
  "sale_price": 200.0,
  "product_type": {
    "code": "FS",
    "description": "Fresco"
  },
  "seller": {
    "id": 1,
    "name": "Comercio",
    "locality": "Florida",
    "province": "Santiago del Estero"
  }
}
```

---

#### 3.6.3 Eliminar un producto

* **Verbo:** `DELETE`

* **Url:**
  `http://localhost:8080/api/v1/products/{idProducto}`
* **Función:** Eliminar un producto siempre que no tenga algún un lote asociado. (Esta es una eliminación física aunque lo mejor sería de forma lógica)..
  ​
* **Tipo de ROL requerido:** `CUSTOMER`
* **Restricción:** El usuario autenticado debe ser el seller del producto
  ​


* **Response Body de ejemplo:**

```json
{
  "message": "Se eliminó con éxito el producto con id: 3"
}
```

---

#### 3.6.4 Visualizar datos de un producto

* **Verbo:** `GET`

* **Url:**
  `http://localhost:8080/api/v1/products/{idProducto}`
* **Función:** Ver los datos de un producto..
  ​
* **Tipo de ROL requerido:** `CUSTOMER`
  ​

* **Response Body de ejemplo:**

```json
{
  "id": 2,
  "product_code": "URU0012555",
  "description": "Chorizo parrillero",
  "width": 0.20,
  "height": 0.30,
  "length": 0.30,
  "net_weight": 1.00,
  "expiration_rate": 10.00,
  "recommended_freezing_temperature": -1.50,
  "freezing_rate": 1.00,
  "sale_price": 250.00,
  "product_type": {
    "code": "FS",
    "description": "Fresco"
  },
  "seller": {
    "id": 1,
    "name": "Comercio",
    "locality": "Florida",
    "province": "Santiago del Estero"
  }
}
```

---

#### 3.6.5 Visualizar lista de productos de un Seller 

* **Verbo:** `GET`

* **Url:**
  `http://localhost:8080/api/v1/products/{idSeller}`
* **Función:** Ver una lista de productos de un seller.
  ​
* **Tipo de ROL requerido:** `CUSTOMER`
  ​

* **Response Body de ejemplo:**

```json
{
  "seller": {
    "id": 1,
    "name": "Comercio",
    "locality": "Florida",
    "province": "Santiago del Estero"
  },
  "products": [
    {
      "id": 1,
      "product_code": "URU0012345",
      "description": "Asado premium",
      "sale_price": 650.00,
      "product_type": {
        "code": "FF",
        "description": "Congelado"
      }
    },
    {
      "id": 2,
      "product_code": "URU0012555",
      "description": "Chorizo parrillero",
      "sale_price": 250.00,
      "product_type": {
        "code": "FS",
        "description": "Fresco"
      }
    }
  ]
}
```

---

## 4 UML

![UML.png](docs/UML_Sprint3.png)

## 5 DER

![DER.png](docs/DER_Sprint3.png)
