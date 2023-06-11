# db-to-config

To read values from your database and populate the configuration array in your `demo.php` file, you can follow these steps:

1. Create a new custom service provider by running the following Artisan command:

```bash
php artisan make:provider demoConfigServiceProvider
```

This will generate a new service provider class in the `app/Providers` directory.

2. Open the generated `demoConfigServiceProvider.php` file and locate the `register` method.

3. Within the `register` method, you can retrieve the values from the database and populate the configuration array using the `Config` facade:

```php
use Illuminate\Support\Facades\Config;
use Illuminate\Support\ServiceProvider;
use App\Models\YourModel; // Replace with the appropriate model

class demoConfigServiceProvider extends ServiceProvider
{
    public function register()
    {
        $configData = YourModel::pluck('value', 'key')->toArray(); // Retrieve data from your model
        
        Config::set('demo', $configData); // Set the 'demo' configuration array with the retrieved data
    }
}
```

Replace `YourModel` with the actual model representing your database table that contains the configuration data. Adjust the code accordingly if you have a different way of retrieving the values from your database.

4. Open the `config/app.php` file and add your custom service provider to the `providers` array:

```php
'providers' => [
    // Other providers...
    App\Providers\demoConfigServiceProvider::class,
],
```

5. Save your changes.

Now, when your Laravel application starts, the `register` method of your `demoConfigServiceProvider` will be called, and it will fetch the configuration values from the database and populate the `demo` configuration array. These values will be accessible throughout your application using the `config('demo')` helper function.

Make sure you have properly set up your database connection details in the `.env` file and configured the model to connect to the correct database table.

Please note that this approach assumes that your database table has two columns: `key` and `value`, where `key` represents the configuration key, and `value` represents the corresponding configuration value. Adjust the code accordingly if your table structure is different.

# If you want to make the $index value as key
If your database doesn't have a separate key column, and you want to set the keys within the service provider while fetching values from the database, you can modify the code as follows:

```php
use Illuminate\Support\Facades\Config;
use Illuminate\Support\ServiceProvider;
use App\Models\YourModel; // Replace with the appropriate model

class demoConfigServiceProvider extends ServiceProvider
{
    public function register()
    {
        $configData = YourModel::all()->pluck('value')->toArray();
        
        $formattedData = [];

        foreach ($configData as $index => $value) {
            $formattedData["demo.$index"] = $value;
        }

        Config::set('demo', $formattedData);
    }
}
```

In this code, we retrieve the values from the database using `pluck('value')` since you don't have a separate key column. We then iterate over the values and generate keys using the `$index` variable as an incremental index. The generated keys are prefixed with `'demo.'`.

Ensure that you replace `YourModel` with the appropriate model representing your database table.

By using this approach, the configuration keys will be dynamically generated within the service provider. The modified configuration array will be accessible throughout your application using the `config('demo')` helper function.

Don't forget to add your custom service provider to the `providers` array in the `config/app.php` file as mentioned in the previous response.

# If you want to create custom key for the values

use Illuminate\Support\Facades\Config;
use Illuminate\Support\ServiceProvider;
use App\Models\YourModel; // Replace with the appropriate model

class DemoConfigServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->booted(function () {
            $configData = YourModel::all()->pluck('id','value')->toArray();

            $formattedData = [];

            foreach ($configData as $value) {
                $key = 'demo_' . strtoupper($this->getFirstWord($value));
                $formattedData[$key] = $value;
            }

            Config::set('demo', $formattedData);
        });
    }

    private function getFirstWord($value)
    {
        $words = explode(" ", $value);
        return $words[0];
    }
}
