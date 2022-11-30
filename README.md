# Install-Magento-Service-Using-Composer

Make sure you have local server install in your device i.e. WAMP or XAMP. Here i'm using WAMP Server

Start localhost server and open database http://localhost/phpmyadmin/.

Create new database "magento2"

Now we need to install Elasticsearch. All vrsion of Magento 2.4.* needs Elasticsearch as search engine.

Check compatibility for <a href='https://experienceleague.adobe.com/docs/commerce-operations/installation-guide/system-requirements.html'>here</a> and download the required version of Elasticsearch. 

I'm using Elasticsearch-7.9 and magento-2.4.2

Extract Zip file under root folder of localhost i.e. C:\wamp64\www\elasticsearch-7.9

Now cd to the extracted directory, and run this command:

<code>C:\wamp64\www\elasticsearch-7.9>bin\elasticsearch.bat</code>

you can check Elasticsearch is properly installed by visiting http://localhost:9200. It should give as it shows in image

![elasticsearch](https://user-images.githubusercontent.com/51017576/204715723-bbebfb7e-7cdd-45d4-9d6e-6bd98e2f403c.png)

Keep Running Elasticsearch and now Download Magento using composer

Open cmd and cd to dir C:\wamp64\www

<code>composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition=2.4.2 <install-directory-name></code>

<code>composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition=2.4.2 magento2</code>

'magento2' is directory name

It will take sometime to download all files and keep an eye on cmd for progress

After complete download Find validateURLScheme function in \vendor\magento\framework\Image\Adapter\Gd2.php and replace it with:

```
private function validateURLScheme(string $filename) : bool
   {
       $allowed_schemes = ['ftp', 'ftps', 'http', 'https'];
       $url = parse_url($filename);
       if ($url && isset($url['scheme']) && !in_array($url['scheme'], $allowed_schemes) && !file_exists($filename)) {
           return false;
       }
       return true;
 }
 ```

Now cd to Magento Directory and run the following command

```
php bin/magento setup:install --base-url="http://localhost/magento2/pub" --db-host="localhost" --db-name="magento2db" --db-user="root" --db-password="" --admin-firstname="admin" --admin-lastname="admin" --admin-email="admin@admin.com" --admin-user="admin" --admin-password="admin123" --language="en_US" --currency="INR" --timezone="Asia/Kolkata" --use-rewrites="1" --backend-frontname="admin" --search-engine="elasticsearch7" --elasticsearch-host="localhost" --elasticsearch-port="9200"
```

Once its install successfully you will be able to see following message

```
Post installation file permissions check…
 For security, remove write permissions from these directories: 'C:\wamp64\www\magento2\app\etc'
 [Progress: 1270 / 1270]
 [SUCCESS]: Magento installation complete.
 [SUCCESS]: Admin Panel URI: /admin
Nothing to import.
```

We need to make necessary changes In app\etc\di.xml, replace Symlink with Copy
```
<virtualType name="developerMaterialization" type="Magento\Framework\App\View\Asset\MaterializationStrategy\Factory">
    <arguments>
        <argument name="strategiesList" xsi:type="array">
            <item name="view_preprocessed" xsi:type="object">Magento\Framework\App\View\Asset\MaterializationStrategy\Symlink</item> !!replace here
            <item name="default" xsi:type="object">Magento\Framework\App\View\Asset\MaterializationStrategy\Copy</item>
        </argument>
    </arguments>
</virtualType>
 ```
 
In vendor\magento\framework\View\Element\Template\File\Validator.php, replace line 138 with:
```
$realPath = str_replace('\\', '/',$this->fileDriver->getRealPath($path));
```

Now Run the following command 
```
php bin/magento indexer:reindex && php bin/magento setup:upgrade && php bin/magento setup:static-content:deploy -f && php bin/magento cache:flush
```

Now you can access Magento Frontend from http://localhost/magento2/pub/ and backend admin from http://localhost/magento2/pub/admin

Disable Magento_TwoFactorAuth 
```
C:\wamp64\www\magento2>php bin/magento module:disable Magento_TwoFactorAuth
```

Lets create custom module and attribute. Download 'learning' directory and place it under C:\wamp64\www\magento2\app\code

Run the following command

```
php bin/magento indexer:reindex && php bin/magento setup:upgrade && php bin/magento setup:static-content:deploy -f && php bin/magento cache:flush
```

You will be able to see custom column as shown in image

![attribute](https://user-images.githubusercontent.com/51017576/204720039-8ed8a395-1418-4d3d-92cd-1e6ddca23862.png)

in frontend you will be able to see like this

![Screenshot_5](https://user-images.githubusercontent.com/51017576/204725909-905d08fe-3e05-47b9-84a7-49415cc297d2.png)


![Screenshot_1](https://user-images.githubusercontent.com/51017576/204721146-84ee3add-7e49-42ce-bf10-2967110a2a1c.png)

Login user: admin and password: admin123 and start adding product

Thank you
