## `Option` ve `Result`
## Neden Option ve Result?
Pek çok dil boş çıktıları temsil edebilmek için **nul/nil/undefined** türlerini ve hataları işlemek için **istisnaları** (Exceptions) kullanır. Rust, özellikle **null işaretçi istisnaları**, gibi istisnalar yoluyla oluşan hassas veri sızıntıları gibi sorunları önlemek için her ikisini de kullanmaz. Bunun yerine Rust, yukarıdaki durumlarla başa çıkmak için `Option` ve `Result` adlarıyla tanımlanmış iki adet genellenmiş `enum` türü sağlar.

> 💭 Önceki bölümlerde, temel olarak [`enum`](https://github.com/rust-lang-tr/dokuman/blob/master/rust-programlama-diline-giris/ikinci-adim/enum.md), [`genellemeler`](https://github.com/rust-lang-tr/dokuman/blob/master/rust-programlama-diline-giris/ikinci-adim/jenerikler.md), [`Option` ve `Result`](https://github.com/rust-lang-tr/dokuman/blob/master/rust-programlama-diline-giris/ikinci-adim/jenerikler.md#genellenmi%C5%9F-enum) türlerine değimniştik.

Hatırlayacağınız gibi: 

  - Bir opsiyonel değer ya içinde bazı değerler barındırır **Some**, ya da hiç bir değere sahip değildir **None**.
  - Bir `result` ise ya başarı bir durumu **Ok**  ya da bir başarısızlığı **Err** temsil edebilir.
 
```Rust
// Bir çıktı, ya bazı değerlere sahip olacak `Some`
// ya da hiçbir değere sahip olmayıp boş `None`olacaktır.
enum Option<T> {
    // T genellenmiş olduğundan herhangi türden olabilir.
    Some(T),
    None,
}

// Bir değerlendirme ise ya başarılı `Ok`
// ya da başarısız `Err` sonucunu temsil edebilir.
enum Result<T, E> {
    // T ve E genellenmiş olduğundan `T` herhangi türden bir değer
    // içerebilirken `E` ise herhangi türden bir hata olabilir.
    Ok(T),
    Err(E),
}
````

💭 Ayrıca [önyükleme kütüphaneleri](https://github.com/rust-lang-tr/dokuman/blob/master/rust-programlama-diline-giris/dorduncu-adim/std-primitives-and-preludes.md#%C3%B6n-y%C3%BCkleme-k%C3%BCt%C3%BCphaneleri) başlığında tartıştığımız gibi, sadece `Option` ve `Result` türleri değil, aynı zamanda onlara ait varyantlar da bu kütüphanelerde bulunur. O nedenle bu türleri programın isim alanlarında bildirmeye gerek kalmadan kullanabiliriz. 

## `Option` türünün temel kullanımları
Bir işlev yahut veri türü yazarken, 

  - Eğer işlevin bir argümanı isteğe bağlıysa,
  - Eğer bir işlevin döndürdüğü değer boş olabiliyorsa,
  - Eğer veri türündeki bir özelliğin değeri boşsa,

Veri türlerini `Option` türü olarak kullanmamız gerekir. 

Örneğin bir işlevin çıktısı hem `&str` hem de boş bir değer döndürebiliyorsa işlevin dönüş değeri `Option<&str>` olarak ayarlanmalıdır:

```Rust
fn secimlik_bir_deger_ver() -> Option<&str> {
    
    // Seçimlik değer boş değilse
    return Some("İşte sana bir değer");
    // Diğer hallerde
    None
}
````

Benzer biçimde bir veri türünün özellik değeri boş veya aşağıdaki örnekte olduğu haliyle `lakap` şeklinde, bildirilmesi zorunlu olmayan, seçimlik bir değer içeriyorsa, o özelliğin `option` olarak ayarlanması gereklidir.

```Rust
struct Kisi {
    onadi:  String,
    lakabi: Option<String>, // Secimlik 
    sonadi: String,
}
````

💭 Hatırlayacağınız gibi ilgili dönüş türünü `(SOme/None)` eşleşme yoluyla yakalamak için örüntü eşleme'den yararlanabiliriz. Örneğin geçerli kullanıcının giriş dizinini öğrenebilmek için `std::env` kütüphanesinde [`home_dir()`](https://doc.rust-lang.org/std/env/fn.home_dir.html) adında artık kullanımdan kalkmış bir metod bulunmaktadır. Haliyle başka sistem kullanıcıları, Linux işletim sisteminde olduğu gibi bir giriş dizinine sahip olamayabileceğinden bu işlev artık [`Option<PathBuf>`](https://doc.rust-lang.org/std/path/struct.PathBuf.html) değeri döndürmektedir.

```Rust
use std::env;

fn main() {
    let dizinim = env::home_dir();
    
    match dizinim {
        Some(yol) => println!("{:?}", yol),
        None      => println!("Başlangıç dizini bulunamadı"),
    }
}
// "/playground"
````

⭐ Ancak, seçimlik argümanları işlevlerle kullanırken, işlevi çağrılarında boş argümanlar için `None` değerlerini de iletmemiz gerekir.

```Rust
// lakap değeri iletilmemiş olabilir
fn tam_adi_getir(onad: &str, sonad: &str, lakap: Option<&str>) -> String {
    match lakap {
        Some(takma) => format!("{} {} {}", takma, onad, sonad),
        None        => format!("{} {}", onad, sonad),
    }
}

fn main() {
    println!("{}", tam_adi_getir("Mehmet", "Ali", None));
    println!("{}", tam_adi_getir("Sinem", "Kırlı", Some("Kuklacı:")));
}
// Mehmet Ali
// Kuklacı: Sinem Kırlı
````
💡 Yukarıdaki örneğin daha iyi bir sürümünde ise, onad, sonad ve lakap alanlarına sahip `Kisi` adında bir yapı oluşturularak tam adı döndüren bir özellik tasarlanabilirdi.

🔎 Bunların haricinde `Option` türleri Rust' taki **nullable işaretçilerle** de kullanılır. Fakat Rust’ ta **null işaretçiler olmadığından** işaretçi türleri geçerli bir konuma işaret etmelidir. O nedenle eğer bir işaretçinin nullable **boş olma olasılığı** bulunuyorsa `Option<Box<T>>` şeklinde bir ifade kullanılır.

## `Result` türünün temel kullanımları
