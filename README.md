# sadis-export-ai
SADIS EXPORT AI Website
const express = require("express");
const cors = require("cors");
const OpenAI = require("openai");

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json({ limit: "20kb" }));

const openai = process.env.OPENAI_API_KEY
  ? new OpenAI({ apiKey: process.env.OPENAI_API_KEY })
  : null;

app.get("/", (req, res) => {
  res.json({
    name: "SADIS EXPORT AI",
    status: "online",
    message: "هوش مصنوعی آماده است."
  });
});

app.get("/api/health", (req, res) => {
  res.json({
    success: true,
    aiConfigured: Boolean(openai)
  });
});

app.post("/api/ai", async (req, res) => {
  try {
    if (!openai) {
      return res.status(503).json({
        success: false,
        error: "کلید API تنظیم نشده است."
      });
    }

    const message = req.body?.message;
    const language = req.body?.language || "fa";

    if (typeof message !== "string" || !message.trim()) {
      return res.status(400).json({
        success: false,
        error: "لطفاً پیام خود را وارد کنید."
      });
    }

    if (message.length > 5000) {
      return res.status(400).json({
        success: false,
        error: "پیام بیش از حد طولانی است."
      });
    }

    const result = await openai.responses.create({
      model: process.env.OPENAI_MODEL || "gpt-4.1-mini",
      instructions:
        "You are SADIS EXPORT AI, a professional business assistant. " +
        "Help with sales, export, marketing, Instagram outreach, and translations. " +
        "Reply in the requested language: " + String(language) + ".",
      input: message.trim()
    });

    res.json({
      success: true,
      answer: result.output_text
    });
  } catch (error) {
    console.error("AI error:", error.message);
    res.status(500).json({
      success: false,
      error: "خطا در دریافت پاسخ هوش مصنوعی."
    });
  }
});

app.listen(PORT, "0.0.0.0", () => {
  console.log("SADIS EXPORT AI server running on port " + PORT);
});
