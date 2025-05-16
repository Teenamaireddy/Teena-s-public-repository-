// === FILE: backend/index.js === const express = require("express"); const cors = require("cors"); const { Configuration, OpenAIApi } = require("openai"); const { Pool } = require("pg"); require("dotenv").config();

const app = express(); app.use(cors()); app.use(express.json());

const configuration = new Configuration({ apiKey: process.env.OPENAI_API_KEY, }); const openai = new OpenAIApi(configuration);

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

app.post("/ask", async (req, res) => { const { question } = req.body;

try { const context = await getSchemaSummary(); const prompt = You are a data analyst. Given the schema:\n${context}\nAnswer the question: ${question};

const completion = await openai.createChatCompletion({
  model: "gpt-4",
  messages: [
    { role: "system", content: "You translate questions to SQL and summarize results." },
    { role: "user", content: prompt },
  ],
});

const response = completion.data.choices[0].message.content;
const sqlMatch = response.match(/```sql([\s\S]*?)```/);
const sql = sqlMatch ? sqlMatch[1].trim() : null;

if (!sql) return res.status(400).json({ error: "No SQL found" });

const result = await pool.query(sql);
return res.json({ sql, data: result.rows, summary: response });

} catch (err) { console.error(err); return res.status(500).json({ error: "Server error" }); } });

async function getSchemaSummary() { const tables = await pool.query(SELECT table_name FROM information_schema.tables  WHERE table_schema = 'public';);

let summary = ""; for (let row of tables.rows) { const columns = await pool.query(SELECT column_name, data_type FROM information_schema.columns  WHERE table_name = '${row.table_name}';); const cols = columns.rows.map(col => ${col.column_name} (${col.data_type})).join(", "); summary += Table ${row.table_name}: ${cols}\n; } return summary; }

const PORT = process.env.PORT || 5000; app.listen(PORT, () => console.log(Server running on ${PORT}));

// === FILE: frontend/src/App.jsx === import React, { useState } from "react"; import axios from "axios"; import { LineChart, Line, XAxis, YAxis, Tooltip, CartesianGrid } from "recharts";

export default function App() { const [question, setQuestion] = useState(""); const [response, setResponse] = useState(null); const [error, setError] = useState(null);

const ask = async () => { setError(null); try { const res = await axios.post("http://localhost:5000/ask", { question }); setResponse(res.data); } catch (err) { setError("Failed to get response"); } };

return ( <div className="p-4 max-w-2xl mx-auto"> <h1 className="text-2xl font-bold mb-4">LLM SQL Analyst</h1> <textarea className="w-full p-2 border rounded" rows="4" value={question} onChange={(e) => setQuestion(e.target.value)} placeholder="Ask a business question..." /> <button onClick={ask} className="mt-2 px-4 py-2 bg-blue-600 text-white rounded"> Ask </button>

{error && <div className="mt-4 text-red-500">{error}</div>}

  {response && (
    <div className="mt-6">
      <h2 className="font-semibold">Answer Summary:</h2>
      <pre className="bg-gray-100 p-2 rounded whitespace-pre-wrap">{response.summary}</pre>

      {response.data.length > 0 && (
        <>
          <h2 className="mt-4 font-semibold">Results:</h2>
          <table className="w-full border mt-2">
            <thead>
              <tr>
                {Object.keys(response.data[0]).map((key) => (
                  <th key={key} className="border p-1">{key}</th>
                ))}
              </tr>
            </thead>
            <tbody>
              {response.data.map((row, i) => (
                <tr key={i}>
                  {Object.values(row).map((val, j) => (
                    <td key={j} className="border p-1">{val}</td>
                  ))}
                </tr>
              ))}
            </tbody>
          </table>

          {Object.keys(response.data[0]).length >= 2 && (
            <LineChart
              width={600}
              height={300}
              data={response.data}
              className="mt-6"
            >
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey={Object.keys(response.data[0])[0]} />
              <YAxis />
              <Tooltip />
              <Line
                type="monotone"
                dataKey={Object.keys(response.data[0])[1]}
                stroke="#8884d8"
              />
            </LineChart>
          )}
        </>
      )}
    </div>
  )}
</div>

); }

// === FILE: db/init.sql === CREATE TABLE sales ( id SERIAL PRIMARY KEY, region TEXT, product TEXT, sales_amount NUMERIC, sales_date DATE, returned BOOLEAN );

INSERT INTO sales (region, product, sales_amount, sales_date, returned) VALUES ('North', 'Widget A', 100.50, '2024-01-10', FALSE), ('South', 'Widget B', 75.00, '2024-01-12', TRUE), ('North', 'Widget A', 110.00, '2024-01-15', FALSE), ('East', 'Widget C', 60.00, '2024-01-20', TRUE), ('West', 'Widget A', 90.00, '2024-02-05', FALSE);

-- === FILE: .env (backend) === OPENAI_API_KEY=your_openai_api_key DATABASE_URL=postgresql://postgres:postgres@localhost:5432/analyticsdb

