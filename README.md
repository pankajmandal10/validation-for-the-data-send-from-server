# validation-for-the-data-send-from-server

Node.js Server

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

//////////////////////////////////////////////////
React Native Client

import React, { useState } from 'react';
import { View, TextInput, Button, Alert } from 'react-native';
import crypto from 'crypto';

const apiUrl = 'http://your-api-url';

const calculateChecksum = (data) => {
  const secretKey = 'yourSecretKey';
  return crypto.createHmac('sha256', secretKey).update(JSON.stringify(data)).digest('hex');
};

const App = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async () => {
    const userData = { username, password };
    const checksum = calculateChecksum(userData);

    try {
      const response = await fetch(`${apiUrl}/api/login`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Checksum': checksum,
        },
        body: JSON.stringify({ data: userData, checksum }),
      });

      const result = await response.json();
      handleResponse(result);
    } catch (error) {
      console.error('Error during login:', error);
    }
  };

  const handleSignup = async () => {
    const userData = { username, password };
    const checksum = calculateChecksum(userData);

    try {
      const response = await fetch(`${apiUrl}/api/signup`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Checksum': checksum,
        },
        body: JSON.stringify({ data: userData, checksum }),
      });

      const result = await response.json();
      handleResponse(result);
    } catch (error) {
      console.error('Error during signup:', error);
    }
  };

  const handleResponse = (response) => {
    const { data, checksum } = response;

    // Verify checksum
    const calculatedChecksum = calculateChecksum(data);

    if (calculatedChecksum !== checksum) {
      Alert.alert('Invalid checksum', 'The response data is not valid.');
      return;
    }

    // Continue processing the response
    Alert.alert('Success', data.message);
  };

  return (
    <View>
      <TextInput
        placeholder="Username"
        value={username}
        onChangeText={(text) => setUsername(text)}
      />
      <TextInput
        placeholder="Password"
        secureTextEntry
        value={password}
        onChangeText={(text) => setPassword(text)}
      />
      <Button title="Login" onPress={handleLogin} />
      <Button title="Signup" onPress={handleSignup} />
    </View>
  );
};

export default App;
