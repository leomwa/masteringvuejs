---
description: >-
  In this section you will create a new list of products and then use a standard
  form layout to support adding and updating products.
---

# Complex Forms

## Create the Products service

In order to support creating, updating and deleting products, add a new `ProductsService`within the **NorthwindService.js** file:

{% code-tabs %}
{% code-tabs-item title="NorthwindService.js" %}
```javascript
...
export const ProductsService = {
    getAll() {
        return apiClient.get('/products')
    },
    get(id) {
        return apiClient.get('/products/' + id)
    },
    create(product) {
        return apiClient.post('/products/', product)
    },
    update(product) {
        return apiClient.put('/products/' + product.id, product)
    },
    delete(id) {
        return apiClient.delete('/products/' + id)
    }
}
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Like the previously created `CategoriesService`, this service simply utilises [Axios](https://github.com/axios/axios) to make HTTP requests against the backend API.

## Creating new components

Start by creating a component to display a list of categories. Within the **views** folder, create a folder named **Products**. Within the new folder, create a file named **ProductList.vue** and implement as follows:

{% code-tabs %}
{% code-tabs-item title="ProductList.vue" %}
```markup
<template>
    <div>
        <h1>Products</h1>
    </div>
</template>

<script>
export default {}
</script>

<style scoped>
</style>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next, create a component to support adding and editing components. Within the **Products** folder, create a new file named **ProductEdit.vue** and implement as follows:

{% code-tabs %}
{% code-tabs-item title="ProductEdit.vue" %}
```markup
<template>
  <div>
    <h1>{{id?`Product #${id}`:'New Product'}}</h1>
  </div>
</template>

<script>
export default {
    props: {
        id: String,
        product: Object
    }
}
</script>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Update the site navigation

Add a Products menu item to the site navigation bar. Open **NavBar.vue** and update as follows:

{% code-tabs %}
{% code-tabs-item title="NavBar.vue" %}
```markup
...
<router-link to="/products" tag="li" class="nav-item" active-class="active">
    <a class="nav-link">Products</a>
</router-link>
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next add routes to support the new components within **router.js**:

{% code-tabs %}
{% code-tabs-item title="router.js" %}
```javascript
...
{
    path: '/products',
    name: 'products',
    component: () => import('./views/Products/ProductList.vue')
},
{
    path: '/products/new',
    name: 'products-new',
    component: () => import('./views/Products/ProductEdit.vue')
},
{
    path: '/products/:id',
    name: 'products-edit',
    component: () => import('./views/Products/ProductEdit.vue'),
    props: true
},
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Save all changes and ensure that you can now navigate to the new products page.

## Displaying the list of products

The next step is to update the `ProductList` component to display the list of products. Start by retrieving the products using the newly created `ProductService`. Within the script block, add the following code:

{% code-tabs %}
{% code-tabs-item title="ProductList.vue" %}
```javascript
...
import { ProductsService } from '@/services/NorthwindService.js'

export default {
    data() {
        return {
            fields: {
                name: { sortable: true },
                unitPrice: { label: 'Price' },
                unitsInStock: { label: 'Stock' },
                actions: {}
            },
            products: []
        }
    },
    created() {
        this.fetchAll()
    },
    methods: {
        fetchAll() {
            ProductsService.getAll()
                .then(result => (this.products= result.data))
                .catch(error => console.error(error))
        }
    }
}
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Then update the template as follows:

{% code-tabs %}
{% code-tabs-item title="ProductList.vue" %}
```markup
...
<template>
  <div>
    <div class="clearfix">
      <h1 class="float-left">Products</h1>
      <router-link tag="button" class="btn btn-primary float-right" :to="{ name: 'products-new' }">
        <i class="fas fa-plus"></i>
      </router-link>
    </div>
    <b-table striped hover :items="products" :fields="fields">
      <template slot="actions" slot-scope="data">
        <router-link tag="button" :to="{ name: 'products-edit', params: { id: data.item.id.toString(), product: data.item } }"
          class="btn btn-secondary btn-sm">
          <i class="fas fa-edit"></i>
        </router-link>
      </template>
    </b-table>
  </div>
</template>
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Save all changes and refresh the site. Ensure that you can view a list of products and that the Add and Edit buttons are wired up correctly

## Adding and editing products

Support for adding and editing products will be implemented within the `ProductEdit` component. When navigating to the component, an `id` property will be included in the query string. If `id` is zero, then the form will be in add mode. If not, the form will be in edit mode.

This component will require a list of categories and suppliers, and access to a specific product \(if editing\). Start by updating **ProductEdit.vue** to import the required services:

{% code-tabs %}
{% code-tabs-item title="ProductEdit.vue" %}
```javascript
...
import {
    CategoriesService,
    SuppliersService,
    ProductsService
} from '@/services/NorthwindService.js'
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next, initialise all required data with default values. Note that `product` is initialised for adding a new product. If in edit mode, this will be replaced by fetching a specific product.

{% code-tabs %}
{% code-tabs-item title="ProductEdit.vue" %}
```javascript
...
data() {
    return {
        categories: [],
        suppliers: [],
        model: {
            id: 0,
            supplierID: null,
            categoryID: null,
            quantityPerUnit: '',
            unitPrice: 0.0,
            unitsInStock: 0,
            unitsOnOrder: 0,
            reorderLevel: 0,
            discontinued: false,
            name: ''
        }
    }
},
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next fetch all categories, suppliers, and a specific product \(if in edit mode\). Using the `created` life cycle event to implement as follows:

{% code-tabs %}
{% code-tabs-item title="ProductEdit.vue" %}
```javascript
...
created() {
    CategoriesService.getAll().then(
        result => (this.categories = result.data)
    )

    SuppliersService.getAll().then(result => (this.suppliers = result.data))

    this.model = this.product || this.model
    if (this.id && !this.product) {
        ProductsService.get(this.id).then(
            result => (this.model = result.data)
        )
    }
}
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now add methods to support saving changes \(either creating or updating\) and navigating back to the list of products:

{% code-tabs %}
{% code-tabs-item title="ProductEdit.vue" %}
```javascript
...
methods: {
    save() {
        if (this.id) {
            ProductsService.update(this.model)
                .then(() => this.navigateBack())
                .catch(error => {
                    console.error(error)
                })
        } else {
            ProductsService.create(this.model)
                .then(() => this.navigateBack())
                .catch(error => {
                    console.error(error)
                })
        }
    },
    navigateBack() {
        this.$router.push('/products')
    }
}
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Finally, add the template to add or edit a product:

{% code-tabs %}
{% code-tabs-item title="ProductEdit.vue" %}
```markup
...
<template>
    <div>
        <h1>{{id?`Product #${id}`:'New Product'}}</h1>

        <form @submit.prevent="save()">
          <div class="form-group">
            <label>Name</label>
            <input type="text" class="form-control" v-model="model.name">
          </div>
          <div class="form-group">
            <label>Category</label>
            <select class="form-control" v-model.number="model.categoryID">
              <option
                v-for="category in categories"
                :key="category.id"
                :value="category.id"
              >{{ category.name }}</option>
            </select>
          </div>
          <div class="form-group">
            <label>Supplier</label>
            <select class="form-control" v-model.number="model.supplierID">
              <option
                v-for="supplier in suppliers"
                :key="supplier.id"
                :value="supplier.id"
              >{{ supplier.companyName }}</option>
            </select>
          </div>
          <div class="form-group">
            <label>Quantity Per Unit</label>
            <input type="text" class="form-control" v-model="model.quantityPerUnit">
          </div>
          <div class="form-row">
            <div class="form-group col-md-4">
              <label>Unit Price</label>
              <input type="number" class="form-control" v-model="model.unitPrice">
            </div>
            <div class="form-group col-md-4">
              <label>Units In Stock</label>
              <input type="number" class="form-control" v-model="model.unitsInStock">
            </div>
            <div class="form-group col-md-4">
              <label>Units On Order</label>
              <input type="number" class="form-control" v-model="model.unitsOnOrder">
            </div>
          </div>
          <div class="form-row">
            <div class="form-group col-md-4">
              <label>Reorder Level</label>
              <input type="number" class="form-control" v-model="model.reorderLevel">
            </div>
            <div class="form-group col-md-4">
              <label>Status</label>
              <div class="form-check">
                <input
                  class="form-check-input"
                  type="checkbox"
                  id="discontinuedCheckbox"
                  v-model="model.discontinued"
                >
                <label class="form-check-label" for="discontinuedCheckbox">Discontinued</label>
              </div>
            </div>
          </div>
    
          <button type="submit" class="btn btn-primary">Save</button>
          <button @click="navigateBack()" class="btn btn-default">Cancel</button>
        </form>
    </div>
</template>
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Deleting products

We still don't have a delete button. You can fix that now. Update **ProductList.vue** as follows.  
Let's add the button below inside the actions template:

{% code-tabs %}
{% code-tabs-item title="ProductList.vue" %}
```markup
...
<button class="btn btn-danger btn-sm" @click="remove(data.item.id)">
    <i class="fas fa-trash-alt"></i>
</button>
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Then add a method to actually delete the product:

{% code-tabs %}
{% code-tabs-item title="ProductList.vue" %}
```javascript
...
remove(id) {
    ProductsService.delete(id)
        .then(() => (this.products = this.products.filter(p => p.id !== id)))
        .catch(error => console.error(error))
}
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Before moving on to the next section, take a moment to ensure that delete behaves as expected.

![](../.gitbook/assets/2019-05-30_12-06-15.gif)

