# PayPal-integration-in-Laravel-10-
Laravel 10 Stripe Payment Gateway  in projects 

refrence code 
https://www.laravelia.com/post/laravel-10-paypal-payment-and-dive-into-details

1st steep 
sub say pehlay apna paypaly devloper per aj k account create kr k PAYPAL_SANDBOX_CLIENT_ID=   ,PAYPAL_SANDBOX_CLIENT_SECRET=
ye lani hogi 

2nd step  ye dono keys apko env main set krni hogi 
PAYPAL_MODE=sandbox
PAYPAL_SANDBOX_CLIENT_ID=
PAYPAL_SANDBOX_CLIENT_SECRET=

3rd step 
apko phir ye command chala k package install krna hoga 
composer require srmklive/paypal:~3.0
and phir isko command ko chalana 
php artisan vendor:publish --provider "Srmklive\PayPal\Providers\PayPalServiceProvider"

4th step 
iss main ap controller bnao ga kisi bhi anmae sa paypal k liye or fir sui  ko route amin use krlna 

5th step route bnana hoga 
// for paypal
Route::controller(PaymentController::class)
    ->prefix('paypal')
    ->group(function () {
        Route::get('payment', 'index')->name('create.payment');
        Route::get('handle-payment', 'handlePayment')->name('make.payment');
        Route::get('cancel-payment', 'paymentCancel')->name('cancel.payment');
        Route::get('payment-success', 'paymentSuccess')->name('success.payment');
    });

6th step main controller ka code ajaye ga 
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Srmklive\PayPal\Services\PayPal as PayPalClient;

class PaymentController extends Controller
{
    public function index() 
    {
        return view('paypal.index');
    }
    public function handlePayment(Request $request)
    {
        $provider = new PayPalClient;
        $provider->setApiCredentials(config('paypal'));
        $paypalToken = $provider->getAccessToken();
        $response = $provider->createOrder([
            "intent" => "CAPTURE",
            "application_context" => [
                "return_url" => route('success.payment'),
                "cancel_url" => route('cancel.payment'),
            ],
            "purchase_units" => [
                0 => [
                    "amount" => [
                        "currency_code" => "USD",
                        "value" => "100.00"
                    ]
                ]
            ]
        ]);
        if (isset($response['id']) && $response['id'] != null) {
            foreach ($response['links'] as $links) {
                if ($links['rel'] == 'approve') {
                    return redirect()->away($links['href']);
                }
            }
            return redirect()
                ->route('cancel.payment')
                ->with('error', 'Something went wrong.');
        } else {
            return redirect()
                ->route('create.payment')
                ->with('error', $response['message'] ?? 'Something went wrong.');
        }
    }

    public function paymentCancel()
    {
        return redirect()
            ->route('create.payment')
            ->with('error', $response['message'] ?? 'You have canceled the transaction.');
    }

    public function paymentSuccess(Request $request)
    {
        $provider = new PayPalClient;
        $provider->setApiCredentials(config('paypal'));
        $provider->getAccessToken();
        $response = $provider->capturePaymentOrder($request['token']);
        if (isset($response['status']) && $response['status'] == 'COMPLETED') {
            return redirect()
                ->route('create.payment')
                ->with('success', 'Transaction complete.');
        } else {
            return redirect()
                ->route('create.payment')
                ->with('error', $response['message'] ?? 'Something went wrong.');
        }
    }
}


abbb isko ap balde main paste krden code jiska name  views/paypal/index.blade.php
 
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Paypal</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/css/bootstrap.min.css"
        integrity="sha384-xOolHFLEh07PJGoPkLv1IbcEPTNtaed2xpHsD9ESMhqIYd0nLMwNLD69Npy4HI+N" crossorigin="anonymous">
    </head>
    <body>
        <div class="panel panel-default">
            <div class="panel-body">
                <h1 class="text-3xl md:text-5xl font-extrabold text-center uppercase mb-12 bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 bg-clip-text text-transparent transform -rotate-2">Make A Payment</h1>
                @if (session()->has('success'))
                    <div class="alert alert-success">
                        {{ session()->get('success') }}
                    </div>
                @endif
                <center>
                    <a href="{{ route('make.payment') }}" class="w-full bg-indigo-500 uppercase rounded-xl font-extrabold text-white px-6 h-8">Pay with PayPal👉</a>
                </center>
            </div>
        </div>
    
    </body>
    </html>
   

