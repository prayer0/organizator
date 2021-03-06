# Organizator

Organizator, geliştiricilerin modüler ve ölçeklenebilir uygulama geliştirmelerine yardımcı olmayı amaçlayan, RequireJS tabanlı ve tarayıcıya yönelik bir JavaScript uygulama çatısıdır.

Kişisel projelerimde kullanılmak üzere geliştirildiği için kod yapısı, klasör düzeni vb. şu an için alışılmadık ve karışıktır.

**DİKKAT:** Kör olma tehlikesi!

## Demo

- [BitProton](https://bitproton.com)
- [SteelDisk](https://steeldisk.com)
- [bitcoinsv-rates.com](http://bitcoinsv-rates.com/)
- https://github.com/bitproton/coldwallet

## Bileşenler

- Routing
- Templating
- Validation
- Database
- Translation

## Kurulum

```
npm install organizator
```

index.html

```
<!DOCTYPE html>
<html>
	</head>
		<title>Organizator Hello World</title>
		<meta charset="utf-8">
	</head>
	<body>
		<script type="text/javascript">
        var require = {
            baseUrl: './node_modules/',
            paths: {
                text: 'organizator/Plugins/text',
                json: 'organizator/Plugins/json',
                route: 'organizator/Plugins/route',
                controller: 'organizator/Plugins/controller',
                xliff: 'organizator/Plugins/xliff',
                css: 'organizator/Plugins/css'
            }
        }
        </script>
		<script data-main="../app.js" src="node_modules/requirejs/require.js"></script>
	</body>
</html>
```

js/app.js

```
require(
    [
        'js/Organizator/Organizator'
    ],
    function(
        Organizator
    ){
        require(
            [
                'route!js/Organizator/Resources/routes',
                'controller!js/Organizator/Resources/controllers'
            ],
            function(
                routes,
                controllers
            ){
                require(
                    [
                        'js/Organizator/Apps/MyApp/MyApp',
                    ],
                    function(
                        MyApp
                    ){
                        new MyApp();
                    }
                );
            }
        );
    }
);
```
js/Organizator/Apps/MyApp.js
```
define(
    [
        'js/Organizator/Organizator/Application'
    ],
    function(
        Organizator_Application
    ){
        class MyApp extends Organizator_Application {
            constructor(){
                super('MyApp');

                this.message = 'hello world'
            }

            hello(){
                document.querySelector('body').insertAdjacentHTML('afterbegin', this.message);
            }
        }
        
        return MyApp;
    }
);
```

## Klasör Yapısı
```
project/
├── index.html
├── css/
│   ├── style.css
└── js/
    ├── app.js
    └── Organizator/
        └── Apps/
            ├── MyApp/
            │   └── MyApp.js
            │   └── view/example.html.njk
            └── AnotherApp/
```

## Genel Bakış 

Organizator'de bir web uygulaması, küçük ve modüler uygulamacıklardan ve bu uygulamacıklar arasındaki iletişimden oluşan bir bütündür. Bir uygulamacık belli bir işlev grubunu üstlenmiş ve require.js modülü olarak tanımlanmış bir sınıftan ibarettir. Örneğin:

js/Organizator/Apps/MyApp.js

```
define(
    [
        'js/Organizator/Organizator/Application'
    ],
    function(
        Organizator_Application
    ){
        class MyApp extends Organizator_Application {
            constructor(){
                super('MyApp');

                this.message = 'hello world'
            }

            hello(){
                document.querySelector('body').insertAdjacentHTML('afterbegin', this.message);
            }
        }
        
        return MyApp;
    }
);
```

Uygulamacıklar bir bootstrap dosyası içinde çağrılır.

js/app.js

```
require(
    [
        'js/Organizator/Organizator'
    ],
    function(
        Organizator
    ){
        require(
            [
                'route!js/Organizator/Resources/routes',
                'controller!js/Organizator/Resources/controllers'
            ],
            function(
                routes,
                controllers
            ){
                require(
                    [
                        'js/Organizator/Apps/MyApp/MyApp',
                        // 'js/Organizator/Apps/AnotherApp/AnotherApp'
                    ],
                    function(
                        MyApp,
                        // AnotherApp
                    ){
                        new MyApp();
                        // new AnotherApp();
                    }
                );
            }
        );
    }
);
```

require.js konfigurasyonu tanımlanır ve bootstrap dosyası require.js ile html içinde çağılır:

```
<!DOCTYPE html>
    <head>
    </head>
    <body>
        <script type="text/javascript">
            var require = {
                baseUrl: './',
                paths: {
                    text: 'js/requirejs/plugins/text',
                    json: 'js/requirejs/plugins/json',
                    route: 'js/requirejs/plugins/route',
                    controller: 'js/requirejs/plugins/controller',
                    xliff: 'js/requirejs/plugins/xliff',
                    css: 'js/requirejs/plugins/css'
                }
            }
        </script>
        <script data-main="js/app.js" src="js/requirejs/require.js"></script>
    </body>
<html>
```


Bir uygulamacığın metotları daha sonra aşağıdaki şekilde çağırılabilir:

```
Organizator.applications.MyApp.hello();
// hello world
```

## Validation (Sınama) Bileşeni

### Sınama Betiği
Organizator HTML ögeleri ve JS objelerini sınamak için küçük bir betik dil kullanır. Bu betik dil aşağıdaki özelliklere sahiptir:

- Parantez kullanarak gruplama,
- ```&&``` (AND) ve ```||``` (OR) operatörleri,
- Parametrik sınamalar için isimlendirilmiş parametreler: ```@length(min=8, max=16)```, ```@email(tld='com')```, ```@email(tld="com")```

Organizator, varsayılan olarak HTML ögelerine tanımlanan org-validate attribute'unu tanır.

```
<input type="text" org-validate="@notblank && ( ( @alphanumeric && @length(min=3) ) || ( @numeric && @length(max=8) ) )"> 
```

Örneğin ```Organizator.Validator.validateHTMLElement(element)``` gibi bir sınama, sınamanın sonucuyla ilgili detaylı bir objeyi döndürür.

### Form Sınama
HTML:
```
<form>
    <input name="username" type="text" org-validate="@alphanumeric && @length(min=4)">
    <input name="email" type="email" org-validate="@email">
    <input name="password" type="password" org-validate="@length=(min=8, max=64)">
    <button type="submit">Submit</button>
</form>
```

JS:
```
let formElement = document.querySelector('form');
let validationResult = Organizator.Validator.validateForm(formElement);

console.log(validationResult);

// {
//     isValid = <boolean>
//     results = <array> // [ItemValidationResult_0, ItemValidationResult_1]
//     errorCount = <int>;
// }
```

### HTML Ögesi Sınama

HTML:
```
<input org-validate="@email && @length(min=3)">
```

JS:
```
let element = document.querySelector('input');
let validationResult = Organizator.Validator.validateHTMLElement(element);

console.log(validationResult);

// {
//     this.item = <HTMLElement>;
//     this.value = <HTMLElement.value>;
//     this.isValid = <boolean>;
//     this.constraints = <object>;
//     this.errorCount = <int>;
// }
```

### JavaScript Objesi Sınama
```
let myObj = {
    foo: 'bar' 
};
let rules = {
    foo: "@length(min=4)"
};

// or

let myObj = {
    foo: 'bar',
    OrganizatorValidationRules: {
        foo: "@length(min=4)"
    }
}

let validationResult = Organizator.Validator.validateObject(myObj, rules);

console.log(validationResult.isValid);
// false
```

### Query String Sınama

```
let queryString = "?foo=bar";
let rules = {
    foo: "@numeric"
};
let validationResult = Organizator.Validator.validateQueryString(queryString, rules);

console.log(validationResult.isValid);
// false
```

### Özel Kısıt Ekleme
Daha sonra istenilen yerde kullanılmak üzere özel bir kısıt sınıfı aşağıdaki şekilde tanımlanabilir:

JS:
```
// js/Organizator/Apps/MyApp/Constraint/MyCustomConstraint.js

define(
    [
        'js/Organizator/Organizator',
        'js/Organizator/Component/Validation/Constraint',
        'js/Organizator/Component/Validation/ConstraintValidationResultBuilder'
    ],
    function(
        Organizator,
        Organizator_Validation_Constraint,
        Organizator_Validation_ConstraintValidationResultBuilder,
    ){ 
        class MyCustomConstraint extends Organizator_Validation_Constraint {
            constructor(options) {
                super();

                Object.assign(this, options);

                this.messages['ERROR_NOT_VALID'] = 'Invalid.';
                this.messages['SUCCESS_VALID'] = 'Valid.';
            }

            static getName(){
                return 'mycustomconstraint';
            }

            validate(value) {
                var resultBuilder = new Organizator_Validation_ConstraintValidationResultBuilder();

                resultBuilder.setValue(value);

                if(value !== this.someoption){
                    resultBuilder.addError(this.messages['ERROR_NOT_VALID']);
                }else{
                    resultBuilder.addSuccess(this.messages['SUCCESS_VALID']);
                }

                return resultBuilder.getResult();
            }
        }
        
        // Register constraint to Organizator on-the-fly
        Organizator.Validator.addConstraint(MyCustomConstraint);
        
        return MyCustomConstraint;
    }
);
```

HTML:
```
<input org-validate="@mycustomconstraint(someoption=3)">
```

## Veritabanı Bileşeni

Veritabanı olarak [LokiJS](https://github.com/techfort/LokiJS) kullanılmıştır. LokiJS MongoDB gibi sorgular yapabileceğiniz hızlı, in-memory bir veritabanıdır.

İki tip veritabanı Organizator ile birlikte dökümana dahil edilir:

- Organizator.Db = geçici veritabanı. sayfa kapandığında tüm veriler silinir.
- Organizator.PersistentDb = kalıcı veritabanı. kullanıcı tarafından silinmediği sürece tüm veriler sayfa tekrar açıldığında erişilebilir.

## Templating Bileşeni

Templating engine olarak Nunjucks kullanılmıştır.

```
{# js/Organizator/Apps/MyApp/view/some-template.html.njk #}

hello world, {{ someparameter }} is here.
```

```
// js/Organizator/Apps/MyApp

define(
    [
        'js/Organizator/Organizator/Application',
        'text!./view/some-template.html.njk' // get the template file
    ],
    function(
        Organizator_Application,
        tpl_someTemplate // give it a name
    ){
        class MyApp extends Organizator_Application {
            constructor(){
                super('MyApp');
            }

            replaceBodyWithSomeHTML(){
                let html = Organizator.Nunjucks.renderString(tpl_someTemplate, { someparameter: 'Organizator'} ); // build html with template + parameters
                document.querySelector('body').innerHTML = html;
            }
        }
        
        return MyApp;
    }
);
```

## Routing Bileşeni

Rotalar js/Organizator/Resources/routes.js altında tanımlanır.

```
// routes.js

define({
    // ..,
    blog_post: {
    path: '/blog/{slug}',
    controller: 'blogPostAction',
    defaults: {
        slug: 'welcome',
        requirements: {
            slug: new RegExp('([a-z])\w+')
        }
    },
    }
})
```

Controller fonksiyonları js/Organizator/Resources/controller.js altında tanımlanır.

```
// controllers.js

define({
    // ..,
    myAction: function(slug, request){
        console.log(slug);
    }
});
```

Örnek bir istek:

```
let request = {
    url: 'http://localhost/blog/my-post',
    mode: 'push', // push, replace, reload, popstate
    popstatable: true,
    absolute: false,
    load: true,
    controller: true
}

Organizator.Routing.request(request)

/* 
 * - sayfa adresi http://localhost/blog/my-post olarak değişir,
 * - myAction fonksiyonu tetiklenir ve konsola 'my-post' çıktısı loglanır.
 */
```

## Build

Uygulama geliştirildikten sonra r.js ya da benzeri bir araç ile ile build edilmelidir.
