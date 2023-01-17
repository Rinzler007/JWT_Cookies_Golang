# JWT_Cookies_Golang

JSON Web Tokens (JWT) and cookies are both used for authentication, but they work in different ways.

JWT is a compact, URL-safe means of representing claims to be transferred between two parties. JWT can be encoded and decoded and the encoded JWT can be passed in the URL, POST parameter or in the HTTP header.

Cookies, on the other hand, are small text files that are stored on a user's browser by a website. They are used to remember a user's preferences, login information, and other data. Cookies are typically sent back and forth between a browser and a server with each request and response.


It is possible to use both cookies and JSON Web Tokens (JWT) together for authentication, but it depends on the specific use case and requirements of the application.

Using cookies in conjunction with JWT can provide an added layer of security. For example, the JWT can be stored in an HttpOnly and Secure cookie on the client-side, which can help protect against cross-site scripting (XSS) and other client-side attacks. The JWT can also be used for access control on the server-side, while the cookie can be used to maintain a session.


Here's an example of how you might implement JSON Web Tokens (JWT) and cookies in a Go (Golang) web application using the "github.com/dgrijalva/jwt-go" library:


```

package main

import (
    "net/http"
    "time"

    jwt "github.com/dgrijalva/jwt-go"
)

var jwtKey = []byte("my_secret_key")

func loginHandler(w http.ResponseWriter, r *http.Request) {
    // Authenticate the user's credentials
    // ...

    // Create a new JWT token
    claims := &jwt.StandardClaims{
        ExpiresAt: time.Now().Add(time.Hour * 24).Unix(),
        Issuer:    "your_issuer",
    }
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    tokenString, _ := token.SignedString(jwtKey)

    // Set the JWT token as a cookie on the client side
    http.SetCookie(w, &http.Cookie{
        Name:    "token",
        Value:   tokenString,
        Expires: time.Now().Add(time.Hour * 24),
    })

    // Return a response to the client
    w.Write([]byte("Logged in!"))
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
    // Get the JWT token from the request headers or cookies
    cookie, _ := r.Cookie("token")
    tokenString := cookie.Value

    // Verify the JWT token
    token, _ := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        return jwtKey, nil
    })

    if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
        // Check that the claims are still valid
        exp := int64(claims["exp"].(float64))
        if exp < time.Now().Unix() {
            http.Error(w, "Token expired", http.StatusUnauthorized)
            return
        }

        // Return a response to the client
        w.Write([]byte("Protected resource"))
    } else {
        http.Error(w, "Invalid token", http.StatusUnauthorized)
    }
}

func main() {
    http.HandleFunc("/login", loginHandler)
    http.HandleFunc("/protected", protectedHandler)
    http.ListenAndServe(":8080", nil)
}

```


