<div align="center">
	<br>
	<br>
	<a href="https://cvmcosta.github.io/ltijs"><img width="360" src="logo-300.svg"></img></a>
</div>



> Turn your application into a fully integratable LTI 1.3 tool provider.

[![travisci](https://img.shields.io/travis/cvmcosta/ltijs.svg)](https://travis-ci.org/Cvmcosta/ltijs)
[![codecov](https://codecov.io/gh/Cvmcosta/ltijs/branch/master/graph/badge.svg)](https://codecov.io/gh/Cvmcosta/ltijs)
[![Node Version](https://img.shields.io/node/v/ltijs.svg)](https://www.npmjs.com/package/ltijs)
[![NPM package](https://img.shields.io/npm/v/ltijs.svg)](https://www.npmjs.com/package/ltijs)
[![NPM downloads](https://img.shields.io/npm/dm/ltijs)](https://www.npmjs.com/package/ltijs)
[![dependencies Status](https://david-dm.org/cvmcosta/ltijs/status.svg)](https://david-dm.org/cvmcosta/ltijs)
[![devDependencies Status](https://david-dm.org/cvmcosta/ltijs/dev-status.svg)](https://david-dm.org/cvmcosta/ltijs?type=dev)
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)
[![APACHE2 License](https://img.shields.io/github/license/cvmcosta/ltijs)](#license)
[![Donate](https://img.shields.io/badge/Donate-Buy%20me%20a%20coffe-blue)](https://www.buymeacoffee.com/UL5fBsi)


Please ⭐️ us on [GitHub](https://github.com/Cvmcosta/ltijs), it always helps! 


> v3.0.0
> MAJOR CHANGE
> - Added Deep Linking Service, exposed through [Provider.DeepLinking](https://cvmcosta.github.io/ltijs/#/deeplinking).
> - Added a new callback, onDeepLinking that tells Ltijs what to display when receiving a deep linking request.
> - Ltik token can now be passed through body parameters, Bearer token authorization header and query parameters.
> - Fixed Database type issue, where some json fields were declared with a `type` key, that changed the type of the entire field.
> - Added the `unique` parameter to the platformUrl field, that deals with the bug that occured when using ltijs in cluster mode with pm2, where platforms would registered more than one time.
> - Added error handling to the registration method. Now it deletes the previously generated key pair kid.


> - View entire [CHANGELOG](changelog.md)

#### Tested on:

| Version | Moodle | Canvas |
| ---- | - | - |
| 3.0 | <center>✔️</center> | <center>✔️</center> |
| 2.5 | <center>✔️</center> | <center>✔️</center> |
| 2.4 | <center>✔️</center> | <center>✔️</center> |
| 2.3 | <center>✔️</center> | <center>✔️</center> |
| 2.1 | <center>✔️</center> | <center>✔️</center> |

<sub>**Previous versions are no longer officially supported*</sub>


## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Tutorial](#tutorial)
- [Usage](#usage)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [License](#license)


---
## Introduction
According to the [IMS Global Learning Consortium](https://www.imsglobal.org/), the Learning Tools Interoperability (LTI) protocol is an IMS standard for integration of rich learning applications within educational environments. <sup>[ref](https://www.imsglobal.org/spec/lti/v1p3/)</sup>


This framework implements a tool provider as an [Express](https://expressjs.com/) server, with preconfigured routes and methods that manage the [LTI 1.3](https://www.imsglobal.org/spec/lti/v1p3/) protocol for you. Making it fast and simple to create a working learning tool without having to worry about manually implementing any of the security and validation required to do so. 


---

## Features

| Feature | Implementation | Documentation |
| --------- | - | - |
| Provider | <center>✔️</center> | <center>✔️</center> |
| [Platform Class](https://cvmcosta.github.io/ltijs/#/platform) | <center>✔️</center> | <center>✔️</center> |
| Database plugins | <center>✔️</center> | <center>✔️</center> |
| [Keyset endpoint support](https://cvmcosta.me/ltijs/#/provider?id=keyset-endpoint) | <center>✔️</center> | <center>✔️</center> |
| [Deep Linking Service Class](https://cvmcosta.me/ltijs/#/deeplinking) | <center>✔️</center> | <center>✔️</center> |
| [Grading Service Class](https://cvmcosta.me/ltijs/#/grading) | <center>✔️</center> | <center>✔️</center> |
| Redis caching | <center></center> | <center></center> |
| Names and Roles Service Class | <center></center> | <center></center> |



---


## Installation

### Installing the package

```shell
$ npm install ltijs
```


### MongoDB
- This package uses mongoDB by default to store and manage the server data, so you need to have it installed, see link bellow for further instructions.
[Installing mongoDB](https://docs.mongodb.com/manual/administration/install-community/)

### PostgreSQL
- This package can also use PosgreSQL to store and manage the server data, it does so through the plugin [ltijs-postgresql](https://www.npmjs.com/package/ltijs-postgresql).

### Firestore
- This package can also use Firestore to store and manage the server data, it does so through the plugin [ltijs-firestore](https://github.com/lucastercas/ltijs-firestore).

***Obs: The officially supported database is MongoDB, the plugins are created by the community and are not mantained by me.***

---


## Tutorial

You can find a quick tutorial on how to set ltijs up and use it to send grades to a Moodle platform [here](https://medium.com/@cvmcosta2/creating-a-lti-provider-with-ltijs-8b569d94825c).

## Quick start


> Install Ltijs

```shell
$ npm install ltijs
```

> Install mongoDB

 - [Installing mongoDB](https://docs.mongodb.com/manual/administration/install-community/)


> Instantiate and use Provider class



```javascript
const path = require('path')

// Require Provider 
const LTI = require('ltijs').Provider

// Configure provider
const lti = new LTI('EXAMPLEKEY', 
            { url: 'mongodb://localhost/database' }, 
            { appUrl: '/', loginUrl: '/login', logger: true })


let setup = async () => {
  // Deploy and open connection to the database
  await lti.deploy()

  // Register platform
  let plat = await lti.registerPlatform({ 
    url: 'https://platform.url',
    name: 'Platform Name',
    clientId: 'TOOLCLIENTID',
    authenticationEndpoint: 'https://platform.url/auth',
    accesstokenEndpoint: 'https://platform.url/token',
    authConfig: { method: 'JWK_SET', key: 'https://platform.url/keyset' }
})

  // Set connection callback
  lti.onConnect((connection, request, response) => {
    // Call redirect function
    lti.redirect(response, '/main')
  })

  // Set main endpoint route
  lti.app.get('/main', (req, res) => {
    // Id token
    console.log(res.locals.token)
    res.send('It\'s alive!')
  })
}
setup()
```

---

## Documentation

See bellow for the complete documentation:

### [Ltijs Documentation](https://cvmcosta.github.io/ltijs/#/provider)

Additional documentations:
   - [Platform class documentation](https://cvmcosta.github.io/ltijs/#/platform)
   - [Deep Linking Service documentation](https://cvmcosta.github.io/ltijs/#/deeplinking)
   - [Grading Service documentation](https://cvmcosta.github.io/ltijs/#/grading)


---

## Contributing

Please ⭐️ us on [GitHub](https://github.com/Cvmcosta/ltijs), it always helps!

If you find a bug or think that something is hard to understand feel free to open an issue or contact me on twitter [@cvmcosta](https://twitter.com/cvmcosta), pull requests are also welcome :)


And if you feel like it, you can donate any amount through paypal, it helps a lot.

[![Donate](https://img.shields.io/badge/Donate-Buy%20me%20a%20coffe-blue)](https://www.buymeacoffee.com/UL5fBsi)

### Contributors

<table>
  <tr>
    <td align="center"><a href="https://github.com/Cvmcosta"><img src="https://avatars2.githubusercontent.com/u/13905368?s=460&v=4" width="100px;" alt="Carlos Costa"/><br /><sub><b>Carlos Costa</b></sub></a><br /><a href="#" title="Code">💻</a><a href="#" title="Answering Questions">💬</a> <a href="#" title="Documentation">📖</a> <a href="#" title="Reviewed Pull Requests">👀</a> <a href="#" title="Talks">📢</a></td>
    <td align="center"><a href="https://github.com/lucastercas"><img src="https://avatars1.githubusercontent.com/u/45924589?s=460&v=4" width="100px;" alt="Lucas Terças"/><br /><sub><b>Lucas Terças</b></sub></a><br /><a href="#" title="Documentation">📖</a> <a href="https://github.com/lucastercas/ltijs-firestore" title="Tools">🔧</a></td>
    <td align="center"><a href="https://github.com/micaelgoms"><img src="https://avatars0.githubusercontent.com/u/23768058?s=460&v=4" width="100px;" alt="Micael Gomes"/><br /><sub><b>Micael Gomes</b></sub></a><br /><a href="#" title="Design">🎨</a></td>    
  
  </tr>
  
</table>






---

## License

[![APACHE2 License](https://img.shields.io/github/license/cvmcosta/ltijs)](LICENSE)
