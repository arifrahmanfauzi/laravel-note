# laravel-note

This is for my personal notes for laravel, sometimes i forgot i

### Form Request

merujuk pada dokumentasi [Validation - Laravel - The PHP Framework For Web Artisans](https://laravel.com/docs/9.x/validation#form-request-validation)

Jika ingin response error validation berupa json, maka harus di tambahkan header *Accept: application/json*

```php
Class LoginRequest extends FormRequest{
    public function authorize()
    {
        return true;
    }

public function rules()
    {
        return [
            //rules for login form
            'email' => 'required',
            'password' => 'required',
        ];
    }  
}
```

### **Custom Validation RequestResponse Message**

this will create a custom response message from FormRequest

```php
class RegisterRequest extends FormRequest
{
    //add this function
     protected function failedValidation(Validator $validator)
    {

        $response = new JsonResponse(['status' =>'failed', 'errors' => $validator->errors()], 200);
        //this will throw new validator exception
        throw new ValidationException($validator, $response);
    }
}
```

Result

```json
{
    "status": "failed",
    "errors": {
        "phone_number": [
            "Phone Number is Required"
        ],
        "email": [
            "email is required"
        ],
        "password": [
            "The password field is required."
        ]
    }
}
```

### **Email Verification**

Implement MustVerifyEmail in User Model

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    // ...
}
```

then create endpoint in api.php

```php
Route::get('email/verify/{id}', 'AuthController@verify')->name('verification.verify'); // Make sure to keep this as your route name

Route::get('email/resend', 'AuthController@resend')->name('verification.resend');
```

in AuthController add method

```php
public function verify($user_id, Request $request) {
    if (!$request->hasValidSignature()) {
        return response()->json(["msg" => "Invalid/Expired url provided."], 401);
    }

    $user = User::findOrFail($user_id);

    if (!$user->hasVerifiedEmail()) {
        $user->markEmailAsVerified();
    }

    return redirect()->to('/');
}

public function resend() {
    if (auth()->user()->hasVerifiedEmail()) {
        return response()->json(["msg" => "Email already verified."], 400);
    }

    auth()->user()->sendEmailVerificationNotification();

    return response()->json(["msg" => "Email verification link sent on your email id"]);
}
```

in register controller add this function

```php
public function register(Request $request): \Illuminate\Http\Response|\Illuminate\Contracts\Foundation\Application|\Illuminate\Contracts\Routing\ResponseFactory
    {
        $validatedData = $request->validate([
            'name' => 'required|max:55',
            'email' => 'email|required|unique:users',
            'password' => 'required|confirmed'
        ]);

        $validatedData['password'] = bcrypt($request->password);
        //this will send email verification notification
        $user = User::create($validatedData)->sendEmailVerificationNotification();

        event(new Registered($user));
        return response(['user' => $user]);
    }
```

### **Laravel Custom Email Notification**

This is some tips if you want to custom email templates.

First things is overwrite method *sendEmailVerificationNotification()* in **User** model.

```php
<?php

class User extends Authenticatable implements MustVerifyEmail 
{
    public function sendEmailVerificationNotification(){
        $this->notify(new VerifyEmailNotification());
    }
}
```

then create a notification class.

`php artisan make:notification VerifyEmailNotification` 

and in VerifyEmailNotification we will extend *VerifyEmail()* .

```php
<?php
class VerifyEmailNotification extends VerifyEmail
{
    public function toMail($notifiable)
    {
    $verificationUrl = $this->verificationUrl($notifiable);

    return (new MailMessage)->view('emails.user.verify', ['url' => $verificationUrl]);
    }
}
```

then create 2 endpoint 

```php
Route::get('email/verify/{id}',[AuthController::class,'verify'])->name('verification.verify');
Route::get('email/resend',[AuthController::class,'resend'])->name('verification.resend');
```

in AuthController.

```php

```

### Api Response

Convert all response to JSON automatically with this middleware

`php artisan make:middleware ForceJsonResponse`

then in that file *ForceJsonResponse.php*

```php
public function handle($request, Closure $next)
{
    $request->headers->set('Accept', 'application/json');
    return $next($request);
}
```

then optional you can add in global *app/Http/Kernal.php* class

```php
'json.response' => \App\Http\Middleware\ForceJsonResponse::class,
```

then you can addet it to the `$middleware` array in *Kernel.php* file

```php
\App\Http\Middleware\ForceJsonResponse::class,
```



### Create mail using markdown format

this is for simple templating mail using markdown template.

first type this command

`php artisan make:mail Invoice --markdown=emails.user.invoice`

that will generate invoice blade template in `resources\views\emails\user\invoice.blade.php` 

you see there [Mail - Laravel - The PHP Framework For Web Artisans](https://laravel.com/docs/9.x/mail#writing-markdown-messages) for available markdown component

and then open *invoice.php* in `app\Mail\Invoice.php` then you can custom email structure like this

```php
<?php

namespace App\Mail;

class Invoice extends Mailable{
    //example static data
    private array $data = ['name' => 'human', 'link' => 'https://google.com','token' => '12d1dwer1yy^$#@'];
     public function build(): static
    {
        return $this->from('example@email.id')->subject('Invoice')->markdown('emails.user.invoice',$this->data);
    }
}
```

and then go to mail markdown template in `resources\views\emails\user\invoice.blade.php` and custom it. 

```php
@component('mail::message')
# Your Invoice
Hello {{ $name }}
The body of your message.

@component('mail::button', ['url' => $link, 'color'=>'primary'])
Button Text
@endcomponent
this is your token : {{$token}}
Thanks,<br>
{{ config('app.name') }}
@component('mail::panel')
    ## This is mail Panel <br>
@endcomponent
@endcomponent

```

then add this code in your controller file.

```php
namespace App\Http\Controllers;

use App\Mail\Invoice;
use Illuminate\Support\Facades\Mail

...

public function sendEmail(){
        Mail::to('example@mail.com')->send(new Invoice());
        return "success";
}
```
