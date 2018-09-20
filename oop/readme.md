# 9 Reasons Why


In this article I am going to ask some questions that could possibly arise during development process.
After that I`ll try to answer them with code examples:

Questions:
-
1. How to not get lost in big project structure with large number of components (500+)?
2. How to create self-documented components?
3. How do you manually test components while developing them?
    This problem is quite important when we want make UI dependent on global state, such as permission handling.
4. How to prepare free UI Kit for your application?
5. How to avoid unexpected bugs during refactoring?
6. How to avoid typo and simplify code refactor?
7. How to make your code more readable and more understandable?
8. How to separate code by responsibility areas?
9. How to improve encapsulation and avoid abstraction leaking?

Answers:
-
All answers will be presented in story like style:

How, in general, do we develop components? - We are coding something in 5-10 minutes range and after we test it manually in browser or electron...
If the component in question is deeply nested, we need to do a couple of manual actions before we're able to reproduce the case.
The situation is exacerbated when hot reload refreshes root component.
All of this increases time, and therefore cost of development.

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

At first we declare interface in demo file , then we implement it as a component:

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
You can tell that our case is different!.
We need imports and global data. For api handling for instance.
For this we can use DI (dependency injection) which allows to swap implementation, therefore keeping components isolated and testable.


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

Other developers can easily know which methods are public for using via $refs, what events are thrown etc. just by looking at demo.
And this documentation is free! Awesome!

Added benefit is that QA guys can easily test completed tasks without the need to reproduce each case. 'staging.com/ProductList.demo.vue'

Project Manager can see the project progress. He can see all demos by visiting route:
'staging.com/demo'.
There will list of all components done in our project. Since PM see progress he can predict terms.
And this is happiness for PM! Profit!

We have 100% covered UI kit of our project! Absolutely free!

### If you like the way i am describing lets continue.

#### What can we do more to have laconic and easy readable and understandable code?
We can use OOP!
Classes give us structured and predictable code. They helps us keep logic in proper predictable place.
Classes are documentation of domain logic. From class (for example: User) we can know the fields and behaviour of user without any documentation.
Since we have strong scheme, we have good ide support (Webstorm, Phpstorm, etc)
We can easy test classes with automated unit testing with frameworks like jest or karma.
IDE can hint properties or methods, it provides auto imports, that can help to avoid typos.

Classes help us work with complicated data structure.
Imagine, we have array with several types of data. And for each type of data we should do different actions.
With classes it is easy!:

```javascript
items.forEach((item)=>{
	if(item instanceof Product){
        item.buy()
    }
  //...
})
```

##### And what helps us write good OOP code in JavaSript?
- Of course TypeScript!

How can it help us?

let's answer another one  first on that:
What is the problem of plain javascript?
Why sometimes there are many unexpected runtime bugs?
 
May be, because we have very unpredictable behaviour of our code.
for example: `product.addDiscount(dicount: 0.2)`
what gonna be if we pass there string? We don\`t - know there is no strict typing in plain js.
Our options to predict app behaviour without running it are severely limited.
And all unpredictable code may cause bugs.

On other hand we have typescript. TypeScript has compiler that throws errors on every unpredictable case.
So we can say compilation is like free testing out of the box!

So here we are. 
#### The main question

The best practice by vuejs documentation for big projects is VUEX.
I would say it's far from truth.

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

##### But you may ask me. 
Wait, but what should I do if I want global state for several components?
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

I experienced migrating from vuex to object driven app while working on my last project (~1300 components)

First we managed our data with vuex. At time we experienced serious trouble with deep nested data, limited modularity, lack of IDE support, reactivity related bugs. And these problems multiplied as project grew.

After switching to object driven storage, most of these issues disappeared altogether


    







