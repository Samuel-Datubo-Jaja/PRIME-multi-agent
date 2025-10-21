FROM --platform=linux/amd64 python:3.12-slim

WORKDIR /app

# Install system dependencies including Node.js (required for MCP servers)
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    gnupg \
    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Verify Node.js installation
RUN node --version && npm --version && npx --version

# Install Python package manager
RUN pip install uv

# Copy project files
COPY pyproject.toml uv.lock ./

# Install Python dependencies including Playwright
RUN uv sync --frozen --no-install-project

# Install Playwright browsers AFTER Python packages are installed
# Use Python's playwright command, not npx
RUN uv run playwright install --with-deps chromium

# Copy application code
COPY *.py ./

# Expose port
EXPOSE 8000

# Run the application
CMD ["uv", "run", "uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8000"]