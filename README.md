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
