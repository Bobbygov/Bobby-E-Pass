# Bobby-E-Pass
const express = require('express');
const bodyParser = require('body-parser');
const sqlite3 = require('sqlite3').verbose();
const QRCode = require('qrcode');
const app = express();
const port = 3000;

app.use(bodyParser.json());
app.use(express.static('public'));

const db = new sqlite3.Database(':memory:');
db.serialize(() => {
    db.run("CREATE TABLE users (id INTEGER PRIMARY KEY AUTOINCREMENT, idNumber TEXT, password TEXT)");
});

app.post('/generate', (req, res) => {
    const { idNumber, password } = req.body;
    db.run("INSERT INTO users (idNumber, password) VALUES (?, ?)", [idNumber, password], function(err) {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        const userId = this.lastID;
        const qrCodeUrl = `http://localhost:${port}/validate/${userId}`;
        QRCode.toDataURL(qrCodeUrl, (err, url) => {
            if (err) {
                return res.status(500).json({ error: err.message });
            }
            res.json({ qrCodeUrl: url });
        });
    });
});

app.get('/validate/:id', (req, res) => {
    const userId = req.params.id;
    db.get("SELECT * FROM users WHERE id = ?", [userId], (err, row) => {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        if (row) {
            res.send(`User ID: ${row.idNumber} is valid.`);
        } else {
            res.send('Invalid QR Code.');
        }
    });
});

app.listen(port, () => {
    console.log(`QR Code App listening at http://localhost:${port}`);
});
