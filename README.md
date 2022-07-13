# laravel-note

This is for my personal notes for laravel, sometimes i forgot i

### Form Request

merujuk pada dokumentasi [laravel form request]([Validation - Laravel - The PHP Framework For Web Artisans](https://laravel.com/docs/9.x/validation#form-request-validation))

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
