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
