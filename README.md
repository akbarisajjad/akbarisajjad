- 👋 Hi, I’m @akbarisajjad
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
akbarisajjad/akbarisajjad is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
بله، می‌توانید این داشبورد را با استفاده از جاوا اسکریپت (Node.js) یا پی‌اچ‌پی بسازید. در اینجا یک مثال ساده از ایجاد یک داشبورد مدیریت ربات تلگرام با استفاده از Node.js و Express برای بک‌اند و React برای فرانت‌اند ارائه می‌شود.

### بک‌اند (Node.js و Express)

#### ایجاد پروژه و نصب وابستگی‌ها:
```bash
mkdir telegram-bot-dashboard
cd telegram-bot-dashboard
npm init -y
npm install express mongoose body-parser jsonwebtoken cors
```

#### `server.js`
```javascript name=server.js
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');
const userRoutes = require('./routes/users');
const botRoutes = require('./routes/bots');

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(bodyParser.json());

app.use('/api/users', userRoutes);
app.use('/api/bots', botRoutes);

mongoose.connect('mongodb://localhost:27017/telegram-bot-dashboard', { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => app.listen(PORT, () => console.log(`Server running on port ${PORT}`)))
    .catch(err => console.log(err));
```

#### `models/User.js`
```javascript name=models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

module.exports = mongoose.model('User', userSchema);
```

#### `models/Bot.js`
```javascript name=models/Bot.js
const mongoose = require('mongoose');

const botSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  token: { type: String, required: true },
  botName: { type: String, required: true },
  botUsername: { type: String, required: true },
});

module.exports = mongoose.model('Bot', botSchema);
```

#### `routes/users.js`
```javascript name=routes/users.js
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const router = express.Router();

router.post('/register', async (req, res) => {
  const { username, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const user = new User({ username, password: hashedPassword });
  await user.save();
  res.status(201).send('User registered');
});

router.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });
  if (!user) return res.status(400).send('Invalid credentials');
  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(400).send('Invalid credentials');
  const token = jwt.sign({ userId: user._id }, 'your_jwt_secret');
  res.json({ token });
});

module.exports = router;
```

#### `routes/bots.js`
```javascript name=routes/bots.js
const express = require('express');
const jwt = require('jsonwebtoken');
const Bot = require('../models/Bot');
const { verifyToken } = require('../middleware/auth');

const router = express.Router();

router.post('/', verifyToken, async (req, res) => {
  const { token, botName, botUsername } = req.body;
  const bot = new Bot({ userId: req.user.userId, token, botName, botUsername });
  await bot.save();
  res.status(201).send('Bot added');
});

router.get('/', verifyToken, async (req, res) => {
  const bots = await Bot.find({ userId: req.user.userId });
  res.json(bots);
});

module.exports = router;
```

#### `middleware/auth.js`
```javascript name=middleware/auth.js
const jwt = require('jsonwebtoken');

const verifyToken = (req, res, next) => {
  const token = req.header('Authorization');
  if (!token) return res.status(401).send('Access denied');
  try {
    const decoded = jwt.verify(token, 'your_jwt_secret');
    req.user = decoded;
    next();
  } catch (err) {
    res.status(400).send('Invalid token');
  }
};

module.exports = { verifyToken };
```

### فرانت‌اند (React)

#### نصب React
```bash
npx create-react-app client
cd client
npm install axios
```

#### `client/src/App.js`
```javascript name=client/src/App.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [token, setToken] = useState('');
  const [botInfo, setBotInfo] = useState([]);
  const [authToken, setAuthToken] = useState('');

  useEffect(() => {
    if (authToken) {
      axios.get('http://localhost:5000/api/bots', { headers: { 'Authorization': authToken } })
        .then(response => setBotInfo(response.data))
        .catch(error => console.error('There was an error fetching the bots!', error));
    }
  }, [authToken]);

  const handleTokenChange = (e) => {
    setToken(e.target.value);
  };

  const handleSaveToken = () => {
    axios.post('http://localhost:5000/api/bots', { token }, { headers: { 'Authorization': authToken } })
      .then(response => {
        setBotInfo([...botInfo, response.data]);
      })
      .catch(error => {
        console.error('There was an error saving the token!', error);
      });
  };

  const handleLogin = () => {
    // Replace with actual login API call
    axios.post('http://localhost:5000/api/users/login', { username: 'test', password: 'test' })
      .then(response => setAuthToken(response.data.token))
      .catch(error => console.error('There was an error logging in!', error));
  };

  return (
    <div className="App">
      <h1>Telegram Bot Management Dashboard</h1>
      <button onClick={handleLogin}>Login</button>
      <input
        type="text"
        value={token}
        onChange={handleTokenChange}
        placeholder="Enter Telegram Bot Token"
      />
      <button onClick={handleSaveToken}>Save Token</button>
      <div>
        {botInfo.map(bot => (
          <div key={bot._id}>
            <h2>{bot.botName}</h2>
            <p>{bot.botUsername}</p>
          </div>
        ))}
      </div>
    </div>
  );
}

export default App;
```

### استقرار

#### استقرار بک‌اند

بک‌اند را به یک سرویس میزبانی مانند Heroku یا AWS مستقر کنید.

#### استقرار فرانت‌اند

فرانت‌اند را به یک سرویس میزبانی استاتیک مانند Vercel یا Netlify مستقر کنید.

این یک طرح کلی برای شروع است. شما می‌توانید با افزودن ویژگی‌های بیشتر و بهبود رابط کاربری، این پروژه را گسترش دهید. اگر به کد خاص یا کمک بیشتری نیاز دارید، به من اطلاع دهید!
