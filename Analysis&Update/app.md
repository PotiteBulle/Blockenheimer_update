## Analyse sur le fichier 'app.js' de l'application Blockenheimer

Fichier d'origine disponible ici [app.js](https://codeberg.org/xormetric/bblock/src/branch/main/src/app.js)

## 1. Stockage d'Informations Sensibles dans `localStorage`

### Problème
Le code stocke des informations sensibles telles que le nom d'utilisateur·ice et le mot de passe dans `localStorage` en clair :
```javascript
localStorage.setItem("handle", id);
localStorage.setItem("password", pass);
```
`localStorage` est accessible via JavaScript, ce qui signifie qu'il peut être consulté par n'importe quel script en cours d'exécution sur la page, rendant ces données vulnérables aux attaques telles que le Cross-Site Scripting (XSS).

### Solution
Iel est préférable de stocker les informations sensibles, telles que les jetons d'authentification et les mots de passe, dans des cookies sécurisés, marqués `HttpOnly`, ou de gérer la session côté serveur. Les cookies `HttpOnly` ne sont pas accessibles via JavaScript, offrant ainsi une meilleure protection.

```javascript
document.cookie = "handle=" + id + "; Secure; HttpOnly";
```

## 2. Exposition de Données Sensibles dans les URLs

### Problème
Le code inclut des données sensibles dans les URLs, comme par exemple :
```javascript
const user = await sha(agent.session.did);
if ((await fetch(URL+"d/"+user)).ok) {
    goError();
}
```
Les URLs peuvent être enregistrées dans l'historique du navigateur, dans les journaux des serveurs et peuvent être exposées sur le réseau, ce qui peut compromettre la sécurité des utilisateur·ices.

### Solution
Au lieu d'envoyer des informations sensibles dans l'URL, iel est recommandé d'envoyer ces données dans le corps de la requête, par exemple dans une requête POST :
```javascript
fetch(URL + "d", {
  method: "POST",
  body: JSON.stringify({ user: user }),
  headers: { "Content-Type": "application/json" }
});
```

## 3. Validation des Mots de Passe Côté Client

### Problème
La validation des mots de passe est effectuée côté client à l'aide d'une simple expression régulière :
```javascript
if (!pass.match(/^[a-zA-Z\d]{4}(-[a-zA-Z\d]{4}){3}$/)) {
    goNotAppPassword();
    return;
}
```
Cette méthode est insuffisante, car iel peut être contournée par un·e attaquant·e ayant accès au code côté client. De plus, iel ne vérifie pas suffisamment la robustesse du mot de passe.

### Solution
Iel est impératif de valider également le mot de passe côté serveur. Cela garantit que le mot de passe respecte les critères de sécurité avant d'être stocké ou traité.

```javascript
// Validation côté serveur (exemple avec Node.js)
app.post('/login', (req, res) => {
  const { password } = req.body;
  if (!password.match(/^[a-zA-Z\d]{4}(-[a-zA-Z\d]{4}){3}$/)) {
    return res.status(400).send('Format du mot de passe invalide');
  }
  // Poursuivre avec l'authentification
});
```

## 4. Utilisation de ressources externes non vérifiées (Ici, il s'agit d'un CDN vérifié, mais imaginons que ce ne soit pas le cas).

### Problème
Le code importe des bibliothèques externes à partir de sources non vérifiées, comme par exemple (précision : la bibliothèque est légitime, mais iel n'a jamais trop de prudence).
```javascript
import { html, render } from 'https://esm.run/lit-html';
```
Si le CDN est compromis, un·e attaquant·e pourrait injecter du code malveillant dans les fichiers importés.

### Solution
Iel est préférable d'utiliser des copies locales des bibliothèques ou de se fier uniquement à des CDNs vérifiés avec des contrôles d'intégrité. Vous pouvez vérifier l'intégrité des ressources en utilisant l'attribut `integrity` dans les balises `<script>`.

```html
<script src="https://cdn.example.com/library.js" integrity="sha384-..." crossorigin="anonymous"></script>
```

## 5. Validation Insuffisante des Entrées Utilisateur·ices

### Problème
Le code ne valide pas suffisamment les entrées des utilisateur·ices, notamment lors de la saisie du nom d'utilisateur·ice et du mot de passe :
```javascript
<input ${ref(loginHandle)} type="text" name="handle" placeholder="handle">
```
Si l'entrée n'est pas validée correctement, cela peut entraîner des attaques telles que l'injection SQL, le XSS, ou d'autres attaques similaires.

### Solution
Iel est important de valider les entrées des utilisateur·ices à la fois côté client et côté serveur afin de s'assurer qu'elles respectent le format attendu. Par exemple, pour les noms d'utilisateur·ice:
```javascript
const isValidHandle = /^[a-zA-Z0-9_]+$/.test(username);
if (!isValidHandle) {
  alert('Format du nom d'utilisateur·ice invalide');
}
```

## 6. Vulnérabilités d'Authentification

### Problème
Le code utilise un système de mot de passe d'application simple pour l'authentification, ce qui peut être vulnérable si le mot de passe est faible :
```javascript
await agent.login({
  identifier: id.replace(/^@/, ""),
  password: pass,
});
```
Si un·e attaquant·e parvient à accéder au mot de passe, il·elle peut compromettre le compte. De plus, la gestion des erreurs n'est pas assez détaillée pour faciliter le débogage ou l'identification des erreurs de connexion.

### Solution
Iel est conseillé de mettre en place un mécanisme d'authentification plus robuste, tel que OAuth ou basé sur des tokens JWT. Iel est également important d'améliorer la gestion des erreurs pour fournir des informations plus détaillées et éviter les attaques par force brute.


```javascript
// Exemple d'authentification JWT côté serveur
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  // Vérifier le nom d'utilisateur·ice et le mot de passe...
  const token = jwt.sign({ userId: user.id }, 'your-secret-key');
  res.json({ token });
});
```

## 7. Hachage des Mots de Passe Côté Client

### Problème
Le code effectue le hachage de l'identifiant utilisateur·ice côté client en utilisant SHA-256 :
```javascript
const hash = await crypto.subtle.digest("SHA-256", data);
```
Bien que le hachage soit une bonne pratique, le hachage des mots de passe ou des données sensibles côté client est risqué, car il·elle peut être intercepté·e par un·e attaquant·e avant d'atteindre le serveur.

### Solution
Le hachage des mots de passe doit être effectué côté serveur, où iel est possible de mieux contrôler et sécuriser le processus. Utilisez des hachages salt et stockez-les de manière sécurisée dans une base de données (et sécurisez-les de manière appropriée).

```javascript
// Exemple de hachage côté serveur avec bcrypt
const bcrypt = require('bcrypt');
const hashedPassword = await bcrypt.hash(password, 10);
```

