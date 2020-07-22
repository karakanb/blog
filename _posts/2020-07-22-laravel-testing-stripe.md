---
title: "testing laravel subscription payments with cashier and stripe"
excerpt: "I should have a base test suite that can run without an internet connection, and it should be super fast. both of them, at the same time."
---

Laravel has been my go-to framework for all of my side projects thanks to its ease of use, but while integrating [Cashier](https://laravel.com/docs/7.x/billing) into [Nana](https://nana-landing.netlify.app/) recently, I have seen that there is no easy way of testing the subscription creation flows when using Cashier. The docs suggest that you do real calls against the Stripe’s test APIs:
> When testing an application that uses Cashier, you may mock the actual HTTP requests to the Stripe API; however, this requires you to partially re-implement Cashier’s own behavior. Therefore, we recommend allowing your tests to hit the actual Stripe API.

This might be good advice for certain cases, and would definitely provide a good way to test the flow end-to-end, and it has some advantages:

* The flow can be tested end-to-end, along with the API contracts and the endpoints in the application.
* The tests have overall better confidence due to being able to test multiple units within a single test case, such as auth, database interactions, Cashier, payment flows, and more.

but it also has its problems:

* There needs to be test data set-up in the Stripe test dashboard, which makes it tricky to replicate in new projects.
* The test data added to Stripe has to stay up-to-date and has to be separately updated along with the tests as needed.
* If the test data doesn’t have to be on the Stripe dashboard, the tests would need to build the test data and clean-up after themselves after every test, which means quite a lot of boilerplate code even for simple tests.
* You can hit the Stripe APIs rate limits if you have many tests, 25 requests per second on test mode [according to Stripe docs](https://stripe.com/docs/rate-limits).
* The tests are going to be slower, making your overall feedback loop slower.

Although some of these disadvantages can be solved by some clever tricks, it is still not the best approach. For my use-case, I would have liked to go with the flow that requires setting up Stripe test data for once and making the tests rely on them; however, Nana is supposed to be a dead-simple starter kit for Laravel, therefore making the users go and set things up on Stripe dashboard one-by-one was not an option. Another option was including a one-time script to set things up on Stripe, but that was also suboptimal since the file would be a dead code once it was executed, and I didn’t want to confuse users. I had to find an alternative.

## Enter Prophecy

For those of you who don’t know [Prophecy](https://github.com/phpspec/prophecy), it is a dead-simple mocking library for PHP. Prophecy allows you to define behaviors and return values for your mock objects easily:
```php
// Create your mock
$mockUser = $this->prophesize(\App\User::class);

// Define the behavior and return value
$mockUser->doSomething()
    ->shouldBeCalled()
    ->willReturn('something done');
```
Prophecy is not very relevant for the trick I’ll show later in the article, it just sets the base for the following examples. In practice, any mocking library should work.

## Handling Subscription Creation

Let’s say that you have a simple pricing structure, and the user picked one of the options and checked out. At this point, you have to create a subscription for the user with the given information.
```php
<?php

namespace App\Http\Controllers;

use App\User;
use Illuminate\Http\Request;

class PlansController extends Controller
{ 
  public function save(Request $request, string $plan)
  {
      /** @var User $user */
      $user = $request->user();
      $token = $request->get('paymentMethodId');
      $couponCode = $request->get('couponCode');

      // Build the subscription object.
      $subscriptionBuilder = $user->newSubscription('default', $plan);
      if ($couponCode) {
          $subscriptionBuilder->withCoupon($couponCode);
      }

      // Create the subscription.
      $subscriptionBuilder->create($token);

      return redirect(route('home'));
  }
}
```

We have simply received the payment-related information from the request, attached the coupon if it exists, and created the subscription. For the sake of simplicity, the validations on the request are not shown here, **please do not use this controller in production.** We are not going to be dealing with the interface, as it is irrelevant at this point.

## Testing the Controller

Now you have your controller ready, and we would like to test it. A naive approach for it would be to build a regular feature test with a user calling the endpoint and watch the subscription get created on Stripe, but as we have mentioned before, we don’t want to make internet calls because we want our tests to be simple and fast at the same time, not only simple or fast.

Here is the test function for our controller:

```php
<?php

namespace Tests\Feature;

use App\User;
use Laravel\Cashier\SubscriptionBuilder;
use Tests\TestCase;

use function route;

class PlansTest extends TestCase
{
    public function testSubscriptionIsSuccessfullyCreatedWithCoupon()
    {
        // Disable the authentication middleware, we don't need it.
        $this->withoutMiddleware([Authenticate::class]);

        // Prepare request variables.
        $paymentMethodId = 'payment-method';
        $couponCode = 'SOME_COUPON';

        // Prepare the subscription builder mock.
        $subscriptionBuilder = $this->prophesize(SubscriptionBuilder::class);
        $subscriptionBuilder->withCoupon($couponCode)
            ->shouldBeCalled();
        $subscriptionBuilder->create($paymentMethodId)
            ->shouldBeCalled();

        // Prepare the user mock
        $user = $this->prophesize(User::class);
        $user->newSubscription('default', 'standard')
            ->shouldBeCalled()
            ->willReturn($subscriptionBuilder->reveal());

        // Perform the request as our user mock.
        $testRoute = route('plans.save', ['plan' => 'standard']);
        $response = $this->actingAs($user->reveal())
            ->post($testRoute, [
                'paymentMethodId' => $paymentMethodId,
                'couponCode'      => $couponCode,
            ]);

        // Assert that everything went smoothly.
        $response->assertRedirect(route('home'));
    }
}
```
As you can see, the test starts with defining the common request variables and moves onto defining the behavior on the subscription mock. The SubscriptionBuilder class is the return value of `$user->subscription()` in our controller class; therefore, we first need to prepare it.

- The controller should attach the coupon, so define the `withCoupon` method behavior.
- The controller should attempt to create the subscription, so define the create method with the argument `$paymentMethodId`.

Once we have the subscription builder ready, we need to attach it to our user object. 
- When the `newSubscription()` method of the `$user` object is called with those string arguments, the mock will return the subscription builder we have just prepared.

At this point, we are done with our mocks, and we can perform the actual request. It is simply a POST request to the subscription creation endpoint, and it includes the request parameters we have defined at the beginning of our function.

The trick here is the method `actingAs` that is called right before performing the POST request. This value will be associated with the `$request->user()` method automatically, which is used by the controller, and allow us to define the behavior on the user exactly like we want, without ever needing to set up anything on the database, no need for factories, and absolutely no need for internet calls.

That’s it. Now you can use this method to mock your Cashier dependencies without doing any external calls and make sure your tests stay super fast.

## Why should I use this method?

Just like any other tech decision, this approach also has its advantages and disadvantages.

Disadvantages of this approach are:
- It requires mocking the behavior of some application internals, such as the `subscription()` method on the User object. If you haven’t setup Cashier properly on the User model, these tests would still pass.
- It doesn’t test the flow end-to-end, which means that if there are other issues in your pipelines, such as sending invalid tokens or coupons, these tests would still pass.
- It requires you to properly define the return values of the methods used throughout the controller.

Even though it has some disadvantages, this method still has a lot of value in it:

* There is no need to test internals of Cashier, just like suggested in [Cashier docs](https://laravel.com/docs/7.x/billing#testing).
> When testing, remember that that Cashier itself already has a great test suite, so you should only focus on testing the subscription and payment flow of your own application and not every underlying Cashier behavior.
* The tests still cover the peripherals of the controller, such as the routing and request variable passing, which is better than just building a controller instance and calling its save method directly.
* It doesn’t require any internet or database connection, and it is **super fast.**
* Since it doesn’t depend on Stripe’s API, there are no risks of rate-limiting or failed network calls.

## Conclusion

Overall, this approach seems to hit the sweet spot for my needs. It covers enough of the application and provides enough value for such a small effort. As next steps, it would be beneficial to have tests that call Stripe API as well, which might be grouped and executed only on master builds or only when needed, these two approaches can easily work together and would also allow you to isolate your problems in case a bug slips in. For the case of Nana, this approach will definitely allow me to cover all the payment/subscription related endpoints with tests so that users can build on top of those features without caring about setting anything up on Stripe.
