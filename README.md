# Overview

This is simple restful APIs help you to build a two-factor-authentication application. It uses a Google two-factor authentication mobile app for the second verification. 

## Installation


```bash
cd folder_name
npm install
```

```bash
npm run dev
```
## Usage

>Create an unverified token for users when they start registering for a new account
```javascript

app.post("/api/register", (req, res) => {
  const id = uuid.v4();
  try {
    const path = `/user/${id}`;
    const temp_secret = speakeasy.generateSecret();
    db.push(path, { id, temp_secret });
    res.json({ id, secret: temp_secret.base32 });
  } catch (e) {
    console.log(e);
    res.status(500).json({ message: "Error generating secret key" });
  }
});

```

>After creating account, we call this endpoint when the user input the code from the mobile app to our application to register for their device. 
```javascript
app.post("/api/validate", (req, res) => {
  // verify in the frontend
  const { userId, token } = req.body;
  try {
    // Retrieve user from database
    const path = `/user/${userId}`;
    const user = db.getData(path);
    console.log({ user });
    const { base32: secret } = user.secret;
    // Returns true if the token matches
    const tokenValidates = speakeasy.totp.verify({
      secret,
      encoding: "base32",
      token,
      window: 2,
    });
    if (tokenValidates) {
      res.json({ validated: true });
    } else {
      res.json({ validated: false });
    }
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: "Error retrieving user" });
  }
});
```
>After creating the users in the database with an unverified token and having user input the code from Google two-factor-authentication app, we will call this endpoint to update the verification status of the user in the database 
```javascript

app.post("/api/verify", (req, res) => {
  const { userId, token } = req.body;
  try {
    // Retrieve user from database
    const path = `/user/${userId}`;
    const user = db.getData(path);
    console.log({ user });
    const { base32: secret } = user.temp_secret;
    const verified = speakeasy.totp.verify({
      secret,
      encoding: "base32",
      token,
    });
    if (verified) {
      // Update user data
      db.push(path, { id: userId, secret: user.temp_secret });
      res.json({ verified: true });
    } else {
      res.json({ verified: false });
    }
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: "Error retrieving user" });
  }
});


```
## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
