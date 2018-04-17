# WEBS6 Werkcollege week 2 demo: Architecture
In deze demo gaan we de onderwerpen uit het werkcollege behandelen.
We gaan een applicatie maken met providers, verschillende modules en routing.

## Starter
In de starter vind je een applicatie die een lijst van 'superhelden' laat zien. 
Deze gaan we refactoren. 

## Providers
Op dit moment staat er in de het component 

1. Open _/config/auth-config.js_.
2. We gaan gebruik maken van passport en willen lokaal inloggen. Importeer daarom:
```javascript
var passport = require("passport");
var LocalStrategy = require('passport-local').Strategy;
```
3. Nu maken we een strategy aan die het mogelijk maakt je username / password mee te geven en die de juiste user dan op het request (voor in de route) zet:<br/>
Merk op dat we done aanroepen en dan de user óf false zetten. Dit staat gelijk aan ingelogd zijn of niet.
```javascript
passport.use('login', new LocalStrategy(function (username, password, done) {
    var user = users[_.findIndex(users, { name: username })];

    if (user && user.password === password) {
        done(null, user);
    } else {
        done(null, false);
    }
}));
``` 
4. We moeten de app nog laten weten dat we passport gebruiken _app.js_:
```javascript
var passport = require('passport')
app.use(passport.initialize());
```
5. Nu kunnen we dit gebruiken in de index route _/routes/index.route.js_. Require passport en gebruik de middleware in de route:
```javascript
var passport = require("passport");

...
router.post("/login", passport.authenticate('login', { session: false }), function(req, res){
	return res.json({ user: req.user });
});
```
5. Nu kunnen we posten naar http://localhost:3000/login en dan krijgen we de user terug met bijvoorbeeld:
```
username: jan
password: jantje
```
6. We hebben de user teruggegeven, maar we willen natuurlijk de token teruggeven zodat de client die elke keer mee kan sturen.<br/>
Nu kunnen we de login uitbreiden (en meteen maar ook even het password weghalen):
```javascript
var jwt = require('jsonwebtoken');
var secret = 'Hier komt jouw secret die niemand kent, die stop je dus niet in je public GIT code, maar in config of iets dergelijks';
...
var payload = { id: user.id, username: user.name };
var token = jwt.sign(payload, secret, { expiresIn: '24h'});

delete user.password;
user.token = token;
done(null, user);
```
7. Bewaar de token die je gekregen hebt (normaal in local storage o.i.d. in de client). Die ga je meegeven aan de overige requests.
8. Nu gaan we er voor zorgen dat users met JWT kunnen authenticeren, dit doen we dus weer in _/config/auth-config.js_ met de JWT strategy:<br />
We halen de token uit de header en we lezen de payload die we eerder hebben gegenereerd uit.
```javascript
var passportJWT = require("passport-jwt");
var ExtractJwt = passportJWT.ExtractJwt;
var JwtStrategy = passportJWT.Strategy;
...
passport.use('jwt', new JwtStrategy({ jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(), secretOrKey: secret }, function(jwt_payload, next) {
  var user = users[_.findIndex(users, {id: jwt_payload.id})];
  if (user) {
    next(null, user);
  } else {
    next(null, false);
  }
}));
```
9. Nu kan je in _/routes/user.route.js_ de middleware toevoegen om via een JWT header te laten authenticeren:<br/>
De user komt op het request te staan en wanneer je geen authenticatie meegeeft zal je deze resultaten niet terugkrijgen.
```javascript
var passport = require('passport');
...
router.get("/me", passport.authenticate('jwt', { session: false }), function (req, res) {
    res.json({ user: req.user });
});
```
10. Wanneer je het request uitvoert zal je zien dat je nu de juiste data terug krijgt.
11. Zo kan je ook de gehele route Illuminati ineenkeer afschermen, dit doe je in de _app.js_:
```javascript
app.use('/illuminati', passport.authenticate('jwt', { session: false }), require('./routes/illuminati.route')());
```
## Autorizatie
Nu we onszelf bekend kunnen maken, kunnen we ook bepaalde delen van onze applicatie afschermen. Dit gaan we doen met **Connect-Roles**.
1. Eerst willen we Connect-Roles enablen. We maken een nieuw object aan en stellen meteen een failure handler.<br />
Open _/config/roles-config.js_ en voeg het volgende in:
```javascript
var ConnectRoles = require('connect-roles');

var roles = new ConnectRoles();

var roles = new ConnectRoles({
    failureHandler: function (req, res, action) {
        res.status(403).json({
            message: 'access denied',
            action: action
        });
    }
});

module.exports = roles;
``` 
2. Nu gaan we in _app.js_ aangeven dat we deze middleware willen gebruiken. <br/>
**Let op: Dit moet na de passport declaratie!**
```javascript
var roles = require('./config/roles-config');
app.use(roles.middleware());
```
3. We gaan de rol admin beschermen, hier maken we een functie voor in _config/roles-config.js_:
```javascript
roles.use('admin', function (req) {
	if(req.user){
		var adminRoleIndex = _.findIndex(req.user.roles, (r) => r == 'admin');
		if(adminRoleIndex >= 0){
			return true;
		};
		// Don't return false, this way we can get into the next checker.
	}
});
```
4. Nu kunnen we deze gebruiken in _/routes/index.route.js_ om de admin route te beschermen.<br />
Daarbij moeten we ook nog even aangeven dat we via JWT willen authenticeren.
```javascript
var roles = require('../config/roles-config');
...
router.get('/admin', roles.is('admin'), function(req, res){
    res.json({ message: 'Success! Je bent een administrator en je mag deze pagina bekijken.' });
});
```
5. Wanneer je een GET uitvoert zal je zien dat je hem niet meer mag benaderen, tenzij je een header meegeeft.
6. De rollen worden van boven naar beneden behandeld, daarom retourneren we geen false. <br />
Als we nu ook een illuminatierol aangeven kunnen we straks zowel met deze als met de adminrol bij de illuminatie:
```javascript
roles.use('illuminati', function (req) {
	if(req.user){
		var illuminatiRoleIndex = _.findIndex(req.user.roles, (r) => r == 'illuminati');
		if(illuminatiRoleIndex >= 0){
			return true;
		};
		// Don't return false, this way we can get into the next checker.
	}
});
```
Pas daarbij de admin functie aan zodat deze geen string meer als eerste parameter verwacht. Deze methode moet wel onder de illuminatirol staan.<br/>
7. Pas _app.js_ zo aan dat de illuminatiroute deze rol gebruikt:
```javascript
app.use('/illuminati', passport.authenticate('jwt', { session: false }), roles.is('illuminati'), require('./routes/illuminati.route'));
```
8. Merk op dat nu alle illuminati bij deze route kunnen, maar ook nog steeds alle admins.
9. Ten slotte willen we nu de _/:id_ binnen illuminatie beperken tot jezelf:
```javascript
var roles = require('../config/roles-config');
...
router.get("/:id", roles.is('self'), function (req, res) {
    res.json(_.filter(users, u => u.id == req.params.id));
});
```
10. Pas dan de config weer aan met de volgende methode (let op: boven de admin):
```javascript
roles.use('self', function(req){
	if(req.user){
		if(req.user.id == req.params.id){
			return true;
		}
	}
});
``` 
11. Nu kan je alleen jezelf als illuminati opvragen, je moet dus de rol hebben en je moet je eigen ID gebruiken (of je moet admin zijn).
## Meer weten?
Kijk voor passport op http://www.passportjs.org <br/>
Kijk voor connect-roles op https://github.com/ForbesLindesay/connect-roles