 
<p>
  <img src="https://github.com/inspetor/slate/blob/master/source/images/logo-color.png" width="200" height="40" alt="Inspetor Logo"> </img> 
</p>

# Inspetor Antifraud
Antrifraud Inspetor library for PHP. 

## Description
Inspetor is an product developed to help your company to avoid fraudulent transactions. This READ ME file should help you to integrate the Inspetor PHP library into your product with a couple steps. 

## Setup Guide
This is the step-by-step Inspetor integration:

### Composer & Packagist
Composer is an application-level package manager for the PHP programming language that provides a standard format for managing dependencies of PHP software and required libraries. It runs from the command line and installs dependencies (e.g. libraries) for an application. You can see Composer documentation [here](https://getcomposer.org/doc/00-intro.md). 

Packagist is the main repository of available PHP packages. It also provides autoload capabilities for libraries that specify autoload information to ease usage of third-party code. The Inspetor package is there as you can see [here](https://packagist.org/packages/inspetor/inspetor-php).

We use the Composer commands to install the Inspetor PHP library hosted into Packagist as below:

``` composer require inspetor/inspetor-php:[version] (e.g. composer require inspetor/inspetor-php:1.2.1) ```

If you're a lucky person and get no errors, the library should be listed in your composer.json 'require' and the library files should be inside your vendor folder. 

### Library setup 
Now you're almost able to call our beautiful library inside your code. But, first, you need to set some **configuration**. To Inspetor avoid your fraud, you only need to provide us **two things**: *"appId"* and *"trackerName"* like that:
```
'inspetor_config' => [
  'trackerName' => cool.name (e.g. company.api),
  'appId'       => unique.number (e.g. 30cdfed3-9f7f-4aaa-b9f1-033c4dbfef58)
]
```

The *"appId"* is an unique identifier that the awesome Inspetor Team will provide you when you start to pay us. The *"trackerName"* is a name that will help us to find your data in our database and we'll provide you a couple of them. Okay, if you did everything right until now, you're really able to call our functions and to begin your fight against fraudulent transactions with us.

We'll gonna code for real now, so we **strongly** recommend you to create an Inspetor class in your code to start our library. There's where you gonna insert Inspetor config you wrote some lines above and retrieve our client. Confusing? Relax, we're kind enough to show you how to do it.

```
<?php

namespace NiceCompany\Inspetor;

use Inspetor\InspetorClient;

class InspetorClass 
{
  ...
    /**
     * Let's instantiate this awesome library! 
     */
    public function __construct()
    {
        $this->inspetor = new InspetorClient($inspetor_config);
    }

    /**
     * @return Inspetor\InspetorClient $client
     */
    public function getClient() {
        return $this->inspetor;
    }
  ...
}
?>
```

Now, wherever you need to call some Inspetor function, you just need to import this Class and _voilà_. 

*P.S.: you can place your config wherever you think is better, just before instantiate InspetorClient or into a config file that you include in this class. It's up to you, but we provided you some tips in Best Practices & Tips section.*

### Library Calls 

I'm supposing you did an amazing job until this moment, so let's move on. It's time to make some calls and track some data. Nice, huh? Here we go. 

If you've already read the [general Inspetor files](), you should be aware of all of Inspetor requests and trackers, so our intention here is just to show you how to use the PHP version of some of them. 

Let's imagine that you want to put a tracker in your *"create transaction"* flow to send some data that the best Antifraud team should analyze and tell you if it's a fraud or not. So, it's intuitive that you need to call the *inspetorSaleCreation* and pass the data of that sale, right? 

Yeah, but we must ask you a little favor. Considering the fraud context, it's possible that not all of your transaction data 
help us to indenfity fraud, so we created a **Model** for each instance we use (remember what is a Model [here]()) that you must build and fill with it's needed. Let's see a snippet.

```
<?php

namespace NiceCompany\SaleFolder;

use NiceCompany\Inspetor\InspetorClass;

class Sale {
  ...
  
  public function someCompanyFunction() {
      // $company_sale is an example object of the company with sale data
      $inspetor_sale = $this->inspetorSaleBuilder($company_sale);

      $this->inspetor = new InspetorClass();

      if($inspetor_sale) {
          $inspetor->getClient()->trackSaleCreation($inspetor_sale);
      } 
  }
 
  public function inspetorSaleBuilder($company_sale) {
      $model = $this->inspetor->getClient()->getInspetorSale();
      
      $model->setId($company_sale->getId());
      $model->setAccountId($company_sale->getUserId());
      $model->setStatus($company_sale->getSaleStatus());
      $model->setIsFraud($company_sale->getFraud());
      $model->set...
      
      return $model;
  }
  ...
}

?>
```

Following this code and assuming you've builded your model with all required parametes (find out each Model's required parameters [here]()), we *someCompanyFunction* run, the Inspetor code inside will send a great object with all we need to know about that sale. Easy? 

We're using an auxiliar function *inspetorSaleBuilder* to build the *Sale Model* but you don't have to do it, or place it where we do here neither. You could set this *inspetorSaleBuilder* inside your *InspetorClass* that we talked about some lines above, for example. More tips in the section Best Practices & Tips.

### Models 

The last snipped was a simple example to show how you should call our library and build one of our models. But now we're gonna talk about all of our Models, hoping you understand that some of them are not tracked it self but it's needed inside others. Take a look! 



***Principal models***:
- **Auth**: model you fill with ***login*** or ***logout*** data. The name came from "*Authentication*".
```
<?php
// Calling an instance of Model
  $inspetor_auth = $inspetor->getInspetorAuth();

// Filling model with company data
  $inspetor_auth->setAccountId("123");
  $inspetor_auth->setAccountEmail("test@email.com");
  $inspetor_auth->setTimestamp(time()); // time() returns unix timestamp
?>
```
- **Account**: model you fill with your ***user*** data. Account has *address* and *billing_address* as two non required values and both are builded with *Address* Model.
```
<?php
// Calling an instance of Model
  $inspetor_account = $inspetor->getInspetorAuth();

// Filling model with company data
  $inspetor_account->setId("123");
  $inspetor_account->setName("Test Name");
  $inspetor_account->setEmail("test@email.com");
  $inspetor_account->setDocument("07206094880"); // CPF
  $inspetor_account->setPhoneNumber("11953891736"); 
  $inspetor_account->setAddress($inspetor_account_address);
  $inspetor_account->setBillingAddress($inspetor_account_billing_address);
  $inspetor_account->setCreationTimestamp(time());
  $inspetor_account->setUpdateTimestamp(time());
?>
```
- **Event**: model you fill with your ***event*** data (e.g. an party or forum info). The *address* is required here, so you **must** instantiate an *Address* Model to an Event.
```
<?php
// Calling an instance of Model
$inspetor_event = $inspetor->getInspetorEvent();

// Filling model with company data
  $inspetor_event->setId("8000");
  $inspetor_event->setName("Name Test");
  $inspetor_event->setDescription("Description Test");
  $inspetor_event->setCreationTimestamp(time());
  $inspetor_event->setUpdateTimestamp(time());
  $inspetor_event->setSessions([
      [
          "id"        => "123",
          "timestamp" => $event_session_date1
      ],
      [
          "id"        => "124",
          "timestamp" => $event_session_date2
      ]
  ]);
  $inspetor_event->setStatus(EVENT::PRIVATE_STATUS);
  $inspetor_event->setCategories(["Category1", "Category2"]);
  $inspetor_event->setAddress($inspetor_event_address);
  $inspetor_event->setUrl("cool-company-event);
  $inspetor_event->setProducerId("123");
  $inspetor_event->setAdminsId(["123", "234"]);
  $inspetor_event->setSeatingOptions(["Pista", "VIP"]);
?>
```
- **PassRecovery**: model that must contain data from a ***password recovery*** or ***password reset*** request of your API.
```
<?php
// Calling an instance of Model
  $inspetor_pass = $inspetor->getInspetorPassRecovery();

// Filling model with company data
  $inspetor_pass->setRecoveryEmail("test@email.com");
  $inspetor_pass->setTimestamp(time());
?>
```
- **Sale**: model that should be filled with the ***sale*** data you have in your API. The sale status has fixed allowed values:
  - "accepted" but you can use the Transfer const like Sale::ACCEPTED_STATUS
  - "rejected" but you can use the Transfer const like Sale::DECLINED_STATUS
  - "pending" but you can use the Transfer const like Sale::PENDING_STATUS
  - "refunded" but you can use the Transfer const like Sale::REFUNDED_STATUS
  - "manual_analysis" but you can use the Transfer const like Sale::MANUAL_ANALYSIS_STATUS
```
<?php
// Calling an instance of Model
  $inspetor_sale = $inspetor->getInspetorSale();

// Filling model with company data
  $inspetor_sale->setId("1234");
  $inspetor_sale->setAccountId("123");
  $inspetor_sale->setStatus("pending");
  $inspetor_sale->setIsFraud(false);
  $inspetor_sale->setCreationTimestamp(time());
  $inspetor_sale->setUpdateTimestamp(time());
  $inspetor_sale->setItems([$inspetor_item1, $inspetor_item2]);
  $inspetor_sale->setPayment($inspetor_payment);
?>
```
- **Transfer**: model you fill with ***transference*** data of an item of your API (e.g. transfer of a ticket). The transfer status has fixed allowed values:
  - "accepted" but you can use the Transfer const like Transfer::ACCEPTED_STATUS
  - "rejected" but you can use the Transfer const like Transfer::REJECTED_STATUS
  - "pending" but you can use the Transfer const like Transfer::PENDING_STATUS
```
<?php
// Calling an instance of Model
  $inspetor_transfer = $inspetor->getInspetorTransfer();

// Filling model with company data
  $inspetor_transfer->setId("123");
  $inspetor_transfer->setCreationTimestamp(time());
  $inspetor_transfer->setUpdateTimestamp(time());
  $inspetor_transfer->setItemId("9000");
  $inspetor_transfer->setSenderAccountId("124");
  $inspetor_transfer->setReceiverEmail("test@email.com");
  $inspetor_transfer->setStatus("accepted");
?>
```



***Auxiliar models***:
- **Address**: this model appears inside Account and Event models and should be filled with data of an ***user*** or an ***event***.
```
<?php
// Calling an instance of Model
  $inspetor_address = $inspetor->getInspetorAddress();

// Filling model with company data
  $inspetor_address->setStreet("Street Security");
  $inspetor_address->setNumber("123");
  $inspetor_address->setZipCode("05511010");
  $inspetor_address->setCity("Test City");
  $inspetor_address->setState("Test State");
  $inspetor_address->setCountry("Test Country");
  $inspetor_address->setLatitude("123");
  $inspetor_address->setLongitude("123");
?>
```
- **CreditCard**: when your API process a payment done with credit card, this model will be used. It should be filled with ***buyer's creditcard*** secure data. We don't hold all information at all.
```
<?php
// Calling an instance of Model
  $inspetor_cc = $inspetor->getInspetorCreditCard();

// Filling model with company data
  $inspetor_cc->setFirstSixDigits("123456");
  $inspetor_cc->setLastFourDigits("1234");
  $inspetor_cc->setHolderName("Holder Name");
  $inspetor_cc->setHolderCpf("07206094880");
?>
```
- **Item**: when someone buy a ***ticket*** for instance, this Model will be instantiate and filled with that ticket data.
```
<?php
// Calling an instance of Model
  $inspetor_item = $inspetor->getInspetorItem();

// Filling model with company data
  $inspetor_item->setId("9000");
  $inspetor_item->setEventId("8000");
  $inspetor_item->setSessionId("124");
  $inspetor_item->setPrice("50.00");
  $inspetor_item->setSeatingOption("PISTA");
  $inspetor_item->setQuantity("2");
?>
```
- **Payment**: this is a Model that holds the ***transaction*** data (e.g. payment method or installments). The payment status has fixed allowed values:
  - "credit_card" but you can use the Payment const like Payment::CREDIT_CARD
  - "boleto" but you can use the Payment const like Payment::BOLETO
  - "other" but you can use the Payment const like Payment::OTHER_METHOD
```
<?php
// Calling an instance of Model
  $inspetor_payment = $inspetor->getInspetorPayment();

// Filling model with company data
  $inspetor_payment->setId("12345");
  $inspetor_payment->setMethod("credit_card");
  $inspetor_payment->setInstallments("1");
  $inspetor_payment->setCreditCard($inspetor_cc);
?>
```

### What you should notice
Not all of the Model's attributes are required but we trully recommend you work around to pass them all. On the other hand, some of them are **super important** and you should pass it correctly. Let's talk about some of them.
 - Sale:
   - ***is_fraud***: it's an attribute that you **must** pass to indicate that a sale is fraudulent or not even if it's something that we're providing to you (as part of postback process).
 - Event:
   - ***sessions***: it's an attribute that you **must** pass even if you don't use the sessions context. If is that the case, you just need to replicate some of you Event attribute, as *event_id* and *event_date* for example.
 - Address:
   - ***almost all fields***: address is **only required** when you try to track an Event, but exists in Account model as well. The tricky is that once you decide to provide an Address, most of the atributes are required and you'll get a lot of Exceptions if try to pass it incomplete.
 - CreditCard: 
   - ***all fields***: all of them are requested once you set that the *payment_method* as "credit_card", so pay attention to that. 
 - Common requests:
   - ***update_timestamp***: some Models have setters and getters to update_timestamp and creation_timestamp as you can see in the general files, but only update_timestamp is really required and should be setted. When is a create request (e.g *trackAccountCreation()*), the *update_timestamp* provided we'll be used as *create_timestamp*. We recommend use of *time()* PHP function that returns an unix timestamp. 

### Best Practices & Tips
Did you think it was easy? Please tell us and feel free to send suggestions [here](), we really would appreciate that. From now on, we decided to tell you some ~~secrets~~ nice practices we discover during development time and should help you with a cleaner integration. 
  - **InspetorClass**: we already told you about that but it's important. Do it! With this class, you don't need to pass our config everytime and creates a layer between our application and yours, where you can, for instance, create funcions as *modelsBuilders* (we've already talked about that too) to keep all builders in one place. Here's a snippet of InspetorClass like that old before but with an example of *builder*.
```
<?php

namespace NiceCompany\Inspetor;

use Inspetor\InspetorClient;

class InspetorClass 
{
  ...
    /**
     * Let's instantiate this awesome library! 
     */
    public function __construct()
    {
        $this->inspetor = new InspetorClient($inspetor_config);
    }

    /**
     * @return Inspetor\InspetorClient $client
     */
    public function getClient() {
        return $this->inspetor;
    }
    
     /**
     * @param array $auth_data
     * @return Inspetor\Model\Auth
     */
     public function inspetorAuthBuilder($auth_data) {
        $inspetor_auth = $this->getClient()
            ->getInspetorAuth();
        $inspetor_auth->setAccountId($auth_data['userId']);
        $inspetor_auth->setAccountEmail($auth_data['userEmail']);
        $inspetor_auth->setTimestamp(time());

        return $inspetor_auth;
    }

  ...
}
?>
```
  - **InspetorServices**: that's another class to help you with Inspetor instantion and is a totally optional feature. But you'll like it. Do you know something about **dependency injection**? What about **Phalcon**? Dude, these are awesome tools to development with PHP, let's talk about it. 
    - *Dependency Injection*: not only in PHP but in *software engineering* at all, that's a technique whereby one object supplies the dependencies of another object. A "dependency" is an object that can be used, for example a service (that's how we do!). And, if I had to explain to 5-years-old, I would quote John Munsch and please, read more about this awesome topic [here](https://www.freecodecamp.org/news/a-quick-intro-to-dependency-injection-what-it-is-and-when-to-use-it-7578c84fa88f/0).: 
    > When you go and get things out of the refrigerator for yourself, you can cause problems. You might leave the door open, you might get something Mommy or Daddy doesn't want you to have. You might even be looking for something we don't even have or which has expired. What you should be doing is stating a need, "I need something to drink with lunch," and then we will make sure you have something when you sit down to eat.
    - *Phalcon*: that's an open source full stack framework for PHP, optimized for high performance. Among all the advantages and features, Phalcon provide us an nice *dependency injection* service. More about Phalcon [here](https://docs.phalconphp.com/3.4/en/introduction). 
    
    Let's see an example: 
```
<?php

namespace NiceCompany\Inspetor;

use Phalcon\DiInterface;

class InspetorServices
{
    /**
     * @SuppressWarnings(PHPMD.StaticAccess)
     * @param \Phalcon\DI\FactoryDefault $dependencyInjector
     *
     * @return void
     */
    public static function load(DiInterface $dependencyInjector)
    {
        $dependencyInjector->set('InspetorClient', [
            'className' => 'Inspetor\InspetorClient',
            'arguments' => [
                [
                    'type' => 'parameter',
                    'value' => $dependencyInjector
                        ->get('config')
                        ->inspetor
                        ->toArray()
                ]
            ]
        ]);

        $dependencyInjector->set('InspetorTracker', [
            'className' => 'NiceCompany\Inspetor\Inspetor',
            'arguments' => [
                [
                    'type' => 'service',
                    'name' => 'InspetorClient'
                ]
            ]
        ]);
    }
}


``` 

### Conclusion 
WOW! It was lovely to work with you, my friend. We trully hope that our instructions were clear and effective. Again, please tell us if we could make something better and contact us [here](). 

Now you're invited to join our army against fraud 'cause ***STEALING IS BULLSHIT***! 

*DPCL (dope cool)*
