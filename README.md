const express = require("express");
const fs = require("fs");
const path = require("path");

const app = express();
const PORT = process.env.PORT || 3000;
const LOG_FILE = path.join(__dirname, "logs.json");

app.use(express.json());
app.use(express.static(path.join(__dirname, "public")));

function getLogs() {
    try {
        return JSON.parse(fs.readFileSync(LOG_FILE, "utf8"));
    } catch {
        return { logs: [] };
    }
}

function saveLogs(data) {
    fs.writeFileSync(
        LOG_FILE,
        JSON.stringify(data, null, 2),
        "utf8"
    );
}

// Получить логи
app.get("/api/logs", (req, res) => {
    res.json(getLogs());
});

// Добавить новый лог
app.post("/api/logs", (req, res) => {
    const {
        type,
        player = "",
        message,
        moderator = ""
    } = req.body;

    if (!type || !message) {
        return res.status(400).json({
            error: "Укажи type и message"
        });
    }

    const data = getLogs();

    const newLog = {
        id: Date.now(),
        time: new Date().toISOString(),
        type,
        player,
        message,
        moderator
    };

    data.logs.unshift(newLog);

    // Храним максимум 500 записей
    data.logs = data.logs.slice(0, 500);

    saveLogs(data);

    res.status(201).json(newLog);
});

// Очистить все логи
app.delete("/api/logs", (req, res) => {
    saveLogs({ logs: [] });

    res.json({
        success: true,
        message: "Логи очищены"
    });
});

app.listen(PORT, () => {
    console.log(`springswortex запущен на порту ${PORT}`);
});
