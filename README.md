# Go Lang e-Ticaret Sepet Rest Api Örneği
### Go dilinde e-ticaret sepetine ürün ekleme, güncelleme, silme ve adet artırımı işlemleri için REST API'leri kullanabiliriz. Örneğin:

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
)

type Product struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Price int    `json:"price"`
}

type Cart struct {
    Products map[int]int `json:"products"`
}

var products []Product
var cart Cart

// `main` metodu, HTTP isteklerini yönlendirecek router'ı oluşturur ve HTTP isteklerini belirtilen portta dinlemeye başlar.
func main() {
    http.HandleFunc("/products", handleProducts)
    http.HandleFunc("/cart", handleCart)

    log.Fatal(http.ListenAndServe(":8080", nil))
}

// `handleProducts` metodu, HTTP isteğine göre ürünleri yöneten diğer yöntemlere yönlendirir.
func handleProducts(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        getProducts(w, r)
    case http.MethodPost:
        addProduct(w, r)
    case http.MethodPut:
        updateProduct(w, r)
    case http.MethodDelete:
        deleteProduct(w, r)
    default:
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }
}

// `getProducts` metodu, mevcut ürünlerin bir listesini döndürür.
func getProducts(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(products)
}

// `addProduct` metodu, istek gövdesinden bir ürün nesnesi alır, bu ürünü ürün listesine ekler ve eklenen ürünü yanıt olarak döndürür.
func addProduct(w http.ResponseWriter, r *http.Request) {
    var product Product
    json.NewDecoder(r.Body).Decode(&product)
    products = append(products, product)
    json.NewEncoder(w).Encode(product)
}

// `updateProduct` metodu, istek gövdesinden bir ürün nesnesi alır ve bu ürünün ID'si ile eşleşen bir ürünü ürün listesinde günceller. Güncellenen ürünü yanıt olarak döndürür. Eşleşen ürün yoksa 404 hata kodu döndürür.
func updateProduct(w http.ResponseWriter, r *http.Request) {
    var product Product
    json.NewDecoder(r.Body).Decode(&product)
    for i, p := range products {
        if p.ID == product.ID {
            products[i] = product
            json.NewEncoder(w).Encode(product)
            return
        }
    }
    w.WriteHeader(http.StatusNotFound)
}

// `deleteProduct` metodu, istek gövdesinden bir ürün nesnesi alır ve bu ürünün ID'si ile eşleşen bir ürünü ürün listesinden siler. Silinen ürünü yanıt olarak döndürür. Eşleşen ürün yoksa 404 hata kodu döndürür.
func deleteProduct(w http.ResponseWriter, r *http.Request) {
    var product Product
    json.NewDecoder(r.Body).Decode(&product)
    for i, p := range products {
        if p.ID == product.ID {
            products = append(products[:i], products[i+1:]...)
            json.NewEncoder(w).Encode(product)
            return
        }
    }
    w.WriteHeader(http.StatusNotFound)
}

// `handleCart` metodu, HTTP isteğine göre sepeti yöneten diğer metotlara yönlendirir.
func handleCart(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        getCart(w, r)
    case http.MethodPost:
        addToCart(w, r)
    case http.MethodPut:
        updateCart(w, r)
    default:
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }
}

// `getCart` metodu, sepet içeriğini döndürür.
func getCart(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(cart)
}

// `addToCart` metodu, istek gövdesinden bir sepet öğesi nesnesi alır ve bu öğeyi sepete ekler. Eğer sepet zaten bu üründen içeriyorsa, sepet öğesinin miktarını günceller. Eklenen veya güncellenen öğeyi yanıt olarak döndürür.
func addToCart(w http.ResponseWriter, r *http.Request) {
    var product Product
    json.NewDecoder(r.Body).Decode(&product)
    cart.Products[product.ID]++
    json.NewEncoder(w).Encode(cart)
}

// `updateCart` metodu, istek gövdesinden bir sepet öğesi nesnesi alır ve bu öğenin ürünü ile eşleşen bir sepet öğesinin miktarını günceller. Güncellenen öğeyi yanıt olarak döndürür. Eşleşen öğe yoksa 404 hata kodu döndürür.
func updateCart(w http.ResponseWriter, r *http.Request) {
    var product Product
    json.NewDecoder(r.Body).Decode(&product)
    cart.Products[product.ID]++
    json.NewEncoder(w).Encode(cart)
}

```
Bu örnek, ürünleri ve sepeti yönetmek için ayrı ayrı HTTP işlemleri sağlar. Ürünler için HTTP GET, POST, PUT ve DELETE yöntemleri kullanılabilirken, sepet için yalnızca HTTP GET, POST ve PUT yöntemleri kullanılabilir. Bu örnek, daha karmaşık bir e-ticaret uygulaması için yeterli değildir. Sadece örnek bir Rest Api kullanımı için hazırlanmıştır.
