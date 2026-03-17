python3 -m venv venv
source venv/bin/activate   # Mac/Linux
# On Windows: venv\Scripts\activate
pip install --upgrade pip
pip install PySide6
brew install ollama
ollama pull llama2:latest
python3 orbi.py
