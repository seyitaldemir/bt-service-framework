# BT API Framework

Basit ve anlaşılabilir kodlarla hızlıca api noktaları tasarlamak için geliştirilmiştir. Api noktaları geliştirmek için node js'de uzman olmanıza gerek yoktur. Temel javascript ve http bilginiz yeterli olacaktır.

BT API, yalnızca MySQL veritabanı ile çalışmaktadır. Hızlı bir başlangıç için projenize klonlayıp app/config.js dosyasını kendi ayarlarınıza göre düzenleyebilirsiniz.

Örnekleri görmek için projeyi indirip examples dizinine gidebilirsiniz.

# Doküman

Bu framework'ü kullanmak için config.js dosyanızı yapılandırın, modules dizinine bir js dosyası ekleyin ve yeni bir route tanımlayın hepsi bu kadar. Artık bir API noktanız var!

## Ayar

app/config.js dosyasını kendi bilgilerinizle düzenleyebilirsiniz.

```javascript
const config = {
    database: {
        host: "localhost",
        user: "root",
        password: "root",
        database: "test",
        port: "3306",
        dateStrings: true,
        charset: 'utf8_turkish_ci',
        typeCast: function castField( field, useDefaultTypeCasting ) {
            if ( ( field.type === "BIT" ) && ( field.length === 1 ) ) {
                var bytes = field.buffer();
                return( bytes[ 0 ] === 1 );
            }
            return( useDefaultTypeCasting() );
        }
    },
    service: { 
        port: 8080 // Servisin çalışacağı port numarası
    },
    access_token: {
        secret: 'de197ca7b8387913da0f5164785b27f3', // Token bilgileri
        life: 2592000
    }
};
```

BT API, modüller ve rotalar olmak üzere 2 sistem üzerine çalışmaktadır.

1. Modüller; birden fazla rotayı barındırarak rotada kullanılacak parametrelerin kurallarını tutar.
2. Rotalar; HTTP isteği geldiğinde ne yapılacağıyla ilgili kuralları tutar.

## 1. Modül

app/modules dizini altında yer alan *.js dosyaları birer modüldür. Modüllerin içine rotalar, parametre kuralları ve modül adı yazılır. (zorunludur)

Aşağıda bir kullanıcı kayıt örneği verilmektedir.

app/modules dizinine gidip bir users.js dosyası oluşturun ve içine aşağıdaki komutları yazın.
```javascript
//Çekirdek dosyamız çağırıldı.
const Route  = require('../../core/routes');
const route  = new Route();

//Modül adı belirleniyor.
route.name('users');

//Parametre kuralları belirleniyor.
route.rules([
    { name: 'username',  validation: 'username'               },
    { name: 'password',  validation: 'password'               },
    { name: 'full_name', validation: 'string|min:5|max:255'   },
    { name: 'email',     validation: 'email'                  }
]);

//Rota ekleniyor.
route.add('add', '/users', 'POST', {
    insert: {
        table: 'users',
        params: ['username', 'password', 'full_name', 'email']
    }
});
```
### 1.1 Modül Adı

Modül adı yalnızca string değer alır ve rota isimlerinin birbiriyle çakışmasını önler. Hata çıktılarında modül ve rota ismi görüntüler. Modül adını bir sınıf, rotaları ise sınıfa ait yöntemler olarak düşünebilirsiniz.

### 1.2 Parametre Kuralları

HTTP ile api noktanıza gönderilen parametrelerle ilgilenir ve bir dizi kuralları işler. Kurallar; en az 2, en fazla 5 farklı elemana sahip olabilir.

**Kural Türleri**

1. **name:** Gelecek parametrenin ismini string olarak belirtir. HTTP ile api noktanıza gelen parametrenin adını buraya belirtmeniz gerekir. (zorunludur)
2. **validation:** Gelen parametrenin doğrulanmasını sağlar. Eğer doğrulama başarısız olursa "There was a validation error" mesajıyla birlikte HTTP 400 yanıt kodunu döndürür.
3. **alias:** Gelen parametrenin route içinde kullanılacak adı. Örneğin HTTP isteğinde "test_id" olarak gelen parametre adını route içinde "id" olarak kullanmak istiyorsanız bu parametreyi kullanabilirsiniz.
4. **default:** Eğer parametre gönderilmemişse varsayılan değer olarak parametreye atanır.
5. **modifier:** Gelen parametrenin değerini değiştirir.

### 1.2.1 Validation

Doğrulama kuralları [Joi](https://joi.dev/api/?v=17.8.3) kitaplığı ile yapılır. Dahili ve harici olmak üzere 2 farklı doğrulama yöntemi vardır. Dahili yöntemler önceden belirlenmiş bir dizi kuralları barındırır ve hazır olarak kullanılabilir. 8 tür dahili doğrulama kuralı vardır.

1. **integer**  : 0 ile 2147483647 arasında integer değer. ``` { ..., validation: 'integer' }```
2. **enum**     : Belirlenen değerlerden herhangi biriyle eşleşip eşleşmediğini kontrol eder. Enum için verilen değer integer ise tırnaksız yazılır string ise çift tırnak içinde yazılır.  ``` { ..., validation: 'enum:["stringDeger", integerDeger]' }```
3. **username** : minimum: 3, maksimum 30 karakter arasında alfanumerik karakter olup olmadığını kontrol eder. ```{ ..., validation: 'username' }```
4. **password** : küçük büyük harfler ve 0'dan 9'a kadar tüm sayılar kullanılabilir. ```{ ..., validation: 'password' }```
5. **email**    : e-posta olup olmadığını kontrol eder. ```{ ..., validation: 'email' }```
6. **varchar**  : 0 - 255 arasında karakter sayısına sahip string olup olmadığını kontrol eder. ```{ ..., validation: 'varchar' }```
7. **datetime** : Y-m-d H:i:s olarak format doğrulaması yapar. ```{ ..., validation: 'datetime' }```
8. **date**     : Y-m-d olarak format doğrulaması yapar. ```{ ..., validation: 'date' }```

Harici yöntemlerden [Joi](https://joi.dev/api/?v=17.8.3) kitaplığında kullanılan kuralları kullanmak için aşağıdaki örneği kontrol edebilirsiniz.

```javascript
//Joi Şeması
const schema = Joi.object({
    username: Joi.string()
        .alphanum()
        .min(3)
        .max(30)
        .required()
});

/*
 * BT API'de kullanımı
 * Kuralların birbirine karışmaması için "|" işareti kullanıyoruz. 
 * Kurallara parametre atamak için ise ":" işaretini kullanıyoruz. 
 * Kurallara parametre ataması yaparken string parametrede çift tırnak,
 * integer parametrede ise tırnaksız kullanım yapıldığını gör ardı etmemek gerekli.
 */
route.rules([
    { ..., validation: 'string|alphanum|min:3|max:30|required' }
]);
```

### 1.2.2 Default

Kuralda belirtilen isimle (name) parametre gelmemişse default içindeki değer parametrenin yerine geçer. 2 farklı kullanım yöntemi vardır.
1. Statik Değer: Fonksiyon harici herhangi bir veri tipinde değer verilebilir. ```{ name: "total", default: 0 }```
2. Callback (Geri Arama): default parametresine fonksiyon ataması yapılır. Verilen fonksiyonun ilk parametresine HTTP isteğiyle gelen tüm parametreler nesne olarak atanır ve "this" ifadesinin kapsamı değiştirilir. Böylelikle fonksiyon yardımıyla gelen parametreye göre default değerin değiştirilmesi sağlanabilir. Ayrıca "this" ifadesiyle getTokenData fonksiyonu kullanılabilir. Eğer gelen istek başlığında token gönderilir ve başarıyla ayrıştırılırsa getTokenData fonksiyonu ayrıştırılan veriyi nesne olarak geri döndürür.
```javascript
/*
 * Aşağıdaki örnekte 2 farklı parametre kuralı tanımlanmış.
 * "start_date" isimli kural date (Y-m-d) formatında olmak zorunda.
 * Eğer parametre gönderilmemişse varsayılan olarak bugünün tarihi atanacak.
 * "finish_date" isimli kural date (Y-m-d) formatında olmak zorunda.
 * Eğer "start_date" parametresi gelen istekte yoksa "finish_date" parametresine bugünün tarihi atanacak.
 */
 
route.rules([
    {
        name: 'start_date',
        validation: 'date',
        default: date()
    },
    {
        name: 'finish_date',
        validation: 'date',
        default: function(params){
            return params.hasOwnProperty('start_date') ? params.finish_date : date();
        }
    }
]);
```

### 1.2.3 Modifier

Bu kural'a callback fonksiyonu verilirse geriye değiştirilen değeri döndürmek zorunludur. Değer döndürülmezse "undefined" olarak tanımlanır. "default" kuralında olduğu gibi modifier'ın da "this" kapsamı değiştirilir ve getTokenData fonksiyonu kullanılabilir. Callback fonksiyonu yerine önceden tanımlanmış olan fonksiyon isimleri de string olarak kullanılabilir. ```{ ..., modifier: 'join' }```

Callback fonksiyonuna 2 parametre gönderilir. 
1. Gelen parametrenin kendisi.
2. Gelen tüm parametreler (nesne halinde)

```javascript
/*
 * Aşağıdaki modifier örneği çalıştırılmıştır.
 */
 
route.rules([
    {
        name: 'start_date',
        validation: 'date',
        modifier: function(start_date, params){
          return convertTime(start_date) < convertTime(date())
            ? date()
            : start_date;
        }
    }
]);
```

## 2. Rota

Gelen HTTP isteğine vereceği yanıtı belirler. Gelen parametreler kurallardan geçtikten sonra rotaya gönderilir. 4 parametresi vardır.

1. **name:** Rotaya benzersiz isim verilir. 
2. **uri:** API noktasının uri adresi verilir.
3. **type:** HTTP method türü yazılır. (GET, POST, PUT, PATCH, DELETE, HEAD)
4. **settings:** Buraya HTTP cevabında gidecek olan verilerin SQL kodu yazılır. SQL kodunda kullanılacak verilerin SQL'de kullanılmadan önce değiştirilmesi gerekiyorsa "request", SQL sonucunda gelen veriler HTTP yanıtı olarak gitmeden önce değiştirilmesi gerekiyorsa "response" callback elemanları oluşturulur. Burada uzun INSERT ve UPDATE SQL kodları daha kısa kodlar kullanarak oluşturulabilir.

### 2.3 Settings

Verinin oluşturulması için gerekli bilgileri alır. "sql", "insert", "update", "request" ve "response" elemanlarını alabilir.

1. **sql:** Adından da anlaşılabileceği gibi içine SQL kodu yazılır. Bu eleman string veya callback fonksiyonu değerleri alabilir. String olarak kullanıyorsak SQL kodumuzda ihtiyacımız olan parametreleri, parametre kurallarına tanımladıktan sonra burada süslü parantez içinde kullanabiliriz. Örneğin id değerine göre kullanıcının bilgilerini getirmek istiyorsak "id" parametresini parametre kurallarına tanıttıktan sonra şöyle bir sql kodu yazmamız gerekir; "SELECT * FROM users WHERE id = {id}". Eğer callback fonksiyonu olarak kullanılacaksa fonksiyonun ilk parametresine gelen HTTP isteğinden tüm parametreler nesne olarak verilir ve callback fonksiyonundan SQL kodu döndürülmesi beklenir.
2. **insert:** Bu eleman uzun insert kodlarınızı kısaltmak için oluşturulmuştur. "table" ve "params" olmak üzere 2 eleman alır. "table" elemanına insert yapılacak tablo adı string olarak yazılır. "params" elemanına ise insert yapılacak verilerin isimleri dizi olarak yazılır. Hatırlamakta fayda var "params" elemanında kullanılacak parametre isimleri parametre kuralların da tanımlamak zorunludur :)
3. **update:** Bu elemanda update kodlarınızı kısaltmak için oluşturulmuştur. "table", "params" ve "where" olmak üzere 3 eleman alır. insert elemanıyla aynı işlemi yapar tek fark bu elemanda "where" elemanı kullanılabilir. "where" elemanına hangi koşulda update olacağını string olarak belirtiriz. 
4. **request:** HTTP isteğiyle gelen parametreleri işlemek için kullanılır. Değer olarak callback fonksiyonu alır, fonksiyonun ilk parametresine istekle gelen tüm parametreler nesne olarak verilir. Fonksiyonun geri döndürdüğü parametreler SQL'de kullanılacak parametrelere gönderilir bu sebeple parametreleri return ile döndürmek önemlidir.
5. **response:** HTTP isteğine yanıt olarak gidecek olan verilerimizi burada işleriz. Değer olarak callback fonksiyonu alır, fonksiyonun ilk parametresine SQL kodunun sonuçları gönderilir. Fonksiyonda return yapılmazsa HTTP yanıtından herhangi bir veri gönderilmez.
