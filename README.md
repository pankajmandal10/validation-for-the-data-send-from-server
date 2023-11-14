# validation-for-the-data-send-from-server

const express = require('express');
const bodyParser = require('body-parser');
const crypto = require('crypto');

const app = express();
const port = 3000;

app.use(bodyParser.json());

const secretKey = 'yourSecretKey';

// Validation Middleware
const validateRequest = (req, res, next) => {
  const { data, checksum } = req.body;

  // Verify checksum
  const hash = crypto.createHmac('sha256', secretKey).update(JSON.stringify(data)).digest('hex');

  if (hash !== checksum) {
    return res.status(400).json({ error: 'Invalid checksum' });
  }

  // Continue with the request
  next();
};

app.use(validateRequest);

// Login Route
app.post('/api/login', (req, res) => {
  // Validate and process login data
  const responseData = { message: 'Login successful' };
  const responseChecksum = crypto.createHmac('sha256', secretKey).update(JSON.stringify(responseData)).digest('hex');

  res.json({ data: responseData, checksum: responseChecksum });
});

// Signup Route
app.post('/api/signup', (req, res) => {
  // Validate and process signup data
  const responseData = { message: 'Signup successful' };
  const responseChecksum = crypto.createHmac('sha256', secretKey).update(JSON.stringify(responseData)).digest('hex');

  res.json({ data: responseData, checksum: responseChecksum });
});

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
