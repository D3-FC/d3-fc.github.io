#9 Reasons Why


In this article i am going to ask some questions that could possibly arise during development process.
After that i`ll try to answer them with code examples:

Questions:
-
1. How to not get lost in big project structure with large number of components (500+)?
2. How to create self-documented components?
3. How do you manually test components when developing them?
    This question is very actual when we need build some changes depending on global state, such as permission handling.
4. How to prepare free UI Kit for your application?
5. How to avoid unexpected bugs during refactoring?
6. How to avoid typo and simplify code refactor?
7. How to make your code more readable and more understandable?
8. How to separate code by responsibility areas?
9. How to improve encapsulation and avoid abstraction leaking?

Answers:
-
All answers will be presented in story like style:

How, in general, do we develop components? -We are coding something in 5-10 minutes range and after we test it manually in browser or electron...
When we need do some changes in deep nested component or this components depends on some global state or permissions we need make some kind "ritual" 
before we see wanted component. The situation is exacerbated by refreshing page when hot reload can`t resolve compilation.
All this factors are increasing development process and final cost of the whole project.

What can we do?
It`s easy!

First of all we want to add route section -`/demo`. We can use outsource library `vue-book`.
This area will be available only on dev or staging builds:

in router.js file:
```JS
    if (process.env.NODE_ENV === 'dev' && process.env.NODE_ENV === 'staging') {
     router.addRoutes(//...)
    }
```
Second. For each component we will have one Demo File. For example:
`ProductList.vue` will have `ProductList.demo.vue`

Structure will be like:

    components
        Product //domain area of products. All product specified files will be there.
            ProductList.vue
            ProductList.demo.vue
            
When component have public interfaces like `props`, `events`, `methods` for $refs access and so on, 
we might want to test them during development process. So we will use them in Demo file.
In general this kind of writing code is pretty similar to `TDD` (Test Driven Development).

First we write interface in demo that we want get in final, after we write implementations:

```HTML
   //ProductList.demo.vue
   <template>
       <div class="demo-container">
           <div class="demo-container__item">
               <product-list @load="handleOnLoad()">
                   
               </product-list>
           </div>
       </div>
   </template>
   
   <script>
       import ProductList from './ProductList.vue'
   
       export default {
           components: {
               ProductList,
           },
           methods: {
            handleOnLoad() {
             console.log('loaded')
            }
           },
       }
   </script>
   
```

Now we can begin writing ProductList.vue and test it on route `/demo/ProductList.demo.vue`
Very nice, right?
What benefits we also have?
This way of designing components leads developer to write decoupled independent from global state components.
But my case is different you say! And you`ll be right.
In generally you may want to use inside this component for example api abstraction layer.
We will provide it from the parent using DI. like this:


```HTML
   //ProductList.demo.vue
   <template>
       <div class="demo-container">
           <div class="demo-container__item">
               <product-list @load="handleOnLoad()">
                   
               </product-list>
           </div>
       </div>
   </template>
   
   <script>
       import ProductList from './ProductList.vue'
       import Shop from '@/api/Shop'
   
       export default {
           components: {
               ProductList,
           },
           provide:{
               api: Shop
           }
           methods: {
            handleOnLoad() {
             console.log('loaded')
            }
           },
       }
   </script>
   
```

```HTML
   //ProductList.vue
   <template>
       <div>
           
       </div>
   </template>
   
   <script>
       import Shop from '@/api/Shop'
   
       export default {
           inject: ['api'],
           methods: {
            async load() {
             console.log(await this.api.getProducts())
            }
           },
       }
   </script>
   
```
So now we can test component without backend by providing mock api abstraction layer.
Backend can have permissions on `GET /api/products` route. Since we have mocks we don`t care.

What else? Since we are writing `Demo First` code style to implement some functionality we have already declared it in demo.
So demo becomes documentation of all internal functionality of the component.

Other developer can easily know which methods are public for using via $refs what events are thrown an other aspects.
And this documentation is free! Awesome!

QA can simple and isolated test the results. 'staging.com/ProductList.demo.vue'

Project Manager ca see the project progress. He can see all demos by visiting route:
'staging.com/demo'.
There will list of all components done in our project. Since PM see progress he can predict terms.
And this is happiness for PM! Profit!

We have 100% covered UI kit of our project! Absolutely free!

*If you like the way i am describing lets continue.*

####What can we do more to have laconic and easy readable and understandable code?

We can use OOP!
Classes give us structured and predictable code. They helps us keep login in proper predictable place.
Classes are documentation of domain logic. From class (for example: User) we can know the fields and behaviour of user without any documentation.
Since we have strong scheme, we have good ide support (Webstorm, Phpstorm, etc)
We can easy test classes with unit acclimatization testing with frameworks like jest or karma.
IDE can hint properties or methods, it provides auto imports, that can help to avoid typos.

Classes helps us work with complicated data structure.
Imagine, we have array with several types of data. And for each data we should do different actions.
With classes it is easy!:

```javascript
items.forEech((item)=>{
	if(item instanceof Product){
                item.buy()
            }
          //...
})
```

#####And what helps us write good OOP code in JavaSript?

- Of course TypeScript!

How can it help us?

To answer this question lets answer first on that:
What is the problem of plain javascript?
Why sometimes there are many unexpected runtime bugs?
 
MB, because we have very unpredictable behaviour of our code.
for example: `product.addDiscount(dicount: 0.2)`
what gonna be if we pass there string? We dont know there is no strict typing in plane js.
We can`t predict what gonna happen before runtime.
And all unpredictable code may cause potential bugs.

On other hand we have typescript. TypeScript has compiler that throws errors on every unpredicted case.
So we can say compilation is like free testing out of the box!

So here we are. 

####The main question

The best practice by vuejs documentation for big projects is VUEX.
But that is the big lie!

VUEX doesnt let us use OOP! One of the most important principle of OOP is `encapsulation`.
Object should mutate his state in its behavior (`methods`)
VUEX is made by FLUX pattern that says: 'we cant mutate the state directly! we should use mutations!'.

So in vuex we should write functional like code style. We have very coupled functions with some state.
This coupling is non semantic. 
The situation is getting worse when one VUEX module using another modules. Now we have `spaghetti`.
We dont have any IDE support. When we want change some module state we dont know what should be broken after.
IDE can`t help us find usage of this peace of code. When we want to delete some module, very hard to find all usages in other components.

In other hand we have classes.
When we want to delete or change some class we can push alt+f7 in webstorm and find all places where this class is used.
Ide helps with renaming and other sugar.

#####But you may ask me. 

Wait, but what should i do if i want global state for several components?
Answer is easy!
we can make separated store like:

theProductListStore.ts
where:

```javascript
export const theProductListStore = {
   products: []
}

```
That`s all! Provide it or import it anywhere you want!




    







