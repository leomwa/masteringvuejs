---
description: >-
  In this topic, we're going to cover how to end-to-end testing from setting it
  up to how to automate
---

# Integration/E2E Testing

If you were brave enough to run `npm run test:e2e` in the overview section, that's what you were expecting to see

![](../.gitbook/assets/e2e-test-run.jpg)

![](../.gitbook/assets/e2e-test-run-1.jpg)

![](../.gitbook/assets/e2e-test-run-2.jpg)

## Creating test case

We're going to create a few meaningful integration tests here just to see if everything is in place. Be aware that this requires the project's dependencies to be running. If you're using a real API or **json-server**, make sure it's running.

Let's create a file called **general.js** under **tests/e2e/specs** and to start with we're going to test 2 things, whether the 404 route is working and if the home page contains the text that's supposed to contains.

{% code-tabs %}
{% code-tabs-item title="general.js" %}
```javascript
describe('General', () => {
    it('404 route should kick off', () => {
        cy.visit('/non-existing-route')
        cy.contains('h1', 'Oops')
    })

    it('home page shows welcome', () => {
        cy.visit('/')
        cy.contains('h1', 'Welcome')
    })
})
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now let's test some CRUD operations under **Suppliers**. This will a bit more complex, we'll test not only UI aspects when navigating to a route, but also whether it's able to connect to the api and successfully update records.

{% code-tabs %}
{% code-tabs-item title="suppliers.js" %}
```javascript
describe('Suppliers', () => {
    beforeEach(() => {
        cy.visit('/suppliers')
        cy.contains('h1', 'Suppliers')
    })

    it('Should open list of suppliers', () => {
        cy.get('table thead th').should('have.length', 4)
    })

    it('Should have 29 suppliers', () => {
        cy.get('table tbody tr').should('have.length', 29)
    })

    it('Should update existing supplier', () => {
        cy.get('table tbody tr:first button:first').click()
        cy.contains('h1', 'Supplier #')
        cy.get('input#companyNameField').type('NEW COMPANY')
        cy.get('button#saveButton').click()

        cy.get('table tbody td:first').should($td => {
            expect($td).to.contain('NEW COMPANY')
        })
    })

    it('Should create new supplier', () => {
        cy.get('button#addSupplier').click()
        cy.get('input#companyNameField').type('NEW COMPANY')
        cy.get('input#contactNameField').type('NEW CONTACT')
        cy.get('input#contactTitleField').type('CONTACT TITLE')
        cy.get('button#saveButton').click()
        cy.get('table tbody tr:last td:first').should($td => {
            expect($td).to.contain('NEW COMPANY')
        })
    })
})

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Once this is created we can run it using the command `npm run test:e2e2`

![](../.gitbook/assets/2019-06-03_20-16-33.gif)

For more information about the integration test API, check their [documentation](https://docs.cypress.io/api/introduction/api.html)

### Getting ready for automation

While opening the UI looks great, that's not ideal for automation. We want everything to run in the console, so let's update the **package.json** file and include a new script

{% code-tabs %}
{% code-tabs-item title="package.json" %}
```javascript
...
"test:e2e:headless": "vue-cli-service test:e2e --headless"
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

By using the flag `--headless`, Cypress is not going to open the whole UI, but run everything in the console like below:

![](../.gitbook/assets/2019-06-03_20-21-37.jpg)

Cool, that's a step forward. But you might now at this point that for the end-to-end tests to work, we need **json-server** to be running. It's alright to run json-server separately when it comes to running tests locally, but if you want to run the tests in an automated way in a continuous integration pipeline, you also need this to be kick off automatically.

For that we're going to leverage an npm package called **npm-run-all** which will run several scripts at once. You can install it by running `npm i npm-run-all` and once it's installed, we'll make changes to the **package.json**

{% code-tabs %}
{% code-tabs-item title="package.json" %}
```javascript
...
"json-server": "json-server db.json --port 3000",
"test:e2e:ci": "npm-run-all -p -r json-server test:e2e:headless"
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now if we run `npm run test:e2e:ci` it should start both **json-server** and the e2e tests.

