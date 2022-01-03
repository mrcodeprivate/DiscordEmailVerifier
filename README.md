# DiscordEmailVerifier
Quickly verify emails on your tokens for completely free using mail.tm's api.

```js
const crypto = require("crypto");
const axios = require("axios")

function email_verify(access) {
  const e = access;
  if (/(.*):(.*)/g.test(e) == false) {
    throw new Error("Invalid Token:Password string")
  }
  
  axios.get("https://api.mail.tm/domains").then(res => {
    const domain = res.data["hydra:member"][0].domain
    const email = crypto.randomBytes(20).toString('hex');

    axios.post("https://api.mail.tm/accounts", {
      "address": `${email}@${domain}`,
      "password": process.env.EMAIL_PASSWORD
    }).then(res => {
      axios.post("https://api.mail.tm/token", {
        ...res.data,
        ...{"password": process.env.EMAIL_PASSWORD}
      }).then(res => {
        axios.patch("https://discord.com/api/v9/users/@me", {
          "email": `${email}@${domain}`,
          "password": e.split(':')[1]
        }, {
          headers: {"authorization": e.substr(0, e.indexOf(':'))}
        }).catch(error => {
          throw new Error("Invalid Token provided, make sure you are using the format (Token:Password) correctly.")
        })

        setTimeout(() => {
          axios.get("https://api.mail.tm/messages", {
            headers: {"authorization": `Bearer ${res.data["token"]}`}
          }).then(res => console.log(res.data))
        }, 3000)
      })
    }).catch(error => {
      console.log(error.response.data)
    })
  })
}

email_verify("YOUR_TOKEN_HERE:YOUR_PASSWORD_HERE")
```

Made by Eric48906™
