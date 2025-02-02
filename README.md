# AI-Powered Product Recommendation System 🚀

A sophisticated product recommendation system using vector embeddings, natural language search, and personalized email recommendations. Built with Redpanda, ClickHouse, and Ollama.

## 🌟 Features

- **Vector-based Product Search**: Semantic search using MixedBread.ai embeddings
- **Natural Language Queries**: Search products using everyday language
- **Personalized Recommendations**: AI-generated product suggestions
- **Automated Email Marketing**: Smart follow-up emails with relevant product recommendations
- **Real-time Processing**: Stream processing with Redpanda Connect
- **Vector Database**: High-performance similarity search with ClickHouse

## 🛠️ Prerequisites

- Docker
- Python 3.8+
- Redpanda Cloud Account
- MailSlurp Account (for email features)
- 16GB+ RAM recommended

## 📦 Installation

1. **Create and activate virtual environment**:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: .\venv\Scripts\activate
```

2. **Install Redpanda CLI**:
```bash
brew install redpanda-data/tap/redpanda  # On macOS
```

3. **Start ClickHouse**:
```bash
docker run -d --name clickhouse-server -p 8123:8123 -p 9000:9000 clickhouse/clickhouse-server
```

4. **Install Ollama and required models**:
```bash
# Install Ollama
curl https://ollama.ai/install.sh | sh

# Pull required models
ollama pull mxbai-embed-large
ollama pull llama3.2
ollama pull llama-guard3
```

5. **Setup project structure**:
```
redpanda-project/
├── data/
│   └── amazon.csv
├── configs/
│   ├── ingest.yaml
│   ├── connect.yaml
│   └── enhanced_search.yaml
└── .env
```

## 🔧 Configuration

1. **Configure Redpanda**:
```bash
rpk cloud login
rpk cloud config init
```

2. **Set environment variables**:
Create `.env` file:
```env
API_KEY=your_mailslurp_key
REDPANDA_BOOTSTRAP_SERVER=your_cluster_address:9092
```

## 🚀 Usage

1. **Data Ingestion**:
```bash
rpk connect run ./configs/ingest.yaml
```

2. **Natural Language Search**:
```bash
SEARCH_QUERY="looking for a gaming phone under 15000" rpk connect run ./configs/enhanced_search.yaml
```

3. **View Results in ClickHouse**:
```bash
docker exec -it clickhouse-server clickhouse-client

# Sample queries
SELECT COUNT(*) FROM sales;
SELECT * FROM llm_emails ORDER BY txn_id DESC LIMIT 1;
```

4. **Web Interface**:
Access ClickHouse UI at: http://localhost:8123/play

## 📊 Vector Search Examples

### Find Similar Products
```sql
WITH target AS (
    SELECT text_embedding, product_name, category
    FROM sales
    WHERE category LIKE '%Electronics%'
    LIMIT 1
)
SELECT DISTINCT
    s.product_name,
    s.category,
    s.discounted_price,
    1 - cosineDistance(s.text_embedding, target.text_embedding) AS similarity,
    target.product_name as compared_to
FROM sales s, target
WHERE s.product_name != target.product_name
ORDER BY similarity DESC
LIMIT 5;
```

### Natural Language Search
```sql
WITH search_text AS (
    SELECT 'looking for a budget phone with good battery life' as query
)
SELECT 
    s.product_name,
    s.category,
    s.discounted_price,
    s.about_product,
    1 - cosineDistance(s.text_embedding, target.text_embedding) as relevance_score
FROM sales s, target
ORDER BY relevance_score DESC
LIMIT 5;
```

## 📝 Notes

- Ensure sufficient RAM for vector operations
- Keep Ollama running in background
- Check ClickHouse logs if queries are slow
- Monitor Redpanda Console for stream processing

## 🔍 Troubleshooting

Common issues and solutions:

1. **Port 4195 already in use**:
```bash
lsof -i :4195
kill -9 <PID>
```

2. **ClickHouse connection issues**:
```bash
docker logs clickhouse-server
```

3. **Model loading errors**:
```bash
ollama list
ollama pull <model_name>
```

## 📚 Resources

- [Redpanda Documentation](https://docs.redpanda.com/)
- [ClickHouse Documentation](https://clickhouse.com/docs)
- [Ollama GitHub](https://github.com/ollama/ollama)

## 📄 License

MIT License - feel free to use this project for learning and development.

## 🤝 Contributing

Contributions welcome! Please read the contributing guidelines before submitting PRs.
