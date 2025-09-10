ARG VARIANT=17
FROM mcr.microsoft.com/vscode/devcontainers/java:0-${VARIANT}

ARG NODE_VERSION="none"
RUN if [ "${NODE_VERSION}" != "none" ]; then su vscode -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi

ARG USER=vscode
VOLUME /home/$USER/.m2
VOLUME /home/$USER/.gradle

ARG JAVA_VERSION=24.0.2-graalce
RUN sudo mkdir /home/$USER/.m2 /home/$USER/.gradle && sudo chown $USER:$USER /home/$USER/.m2 /home/$USER/.gradle
RUN bash -lc '. /usr/local/sdkman/bin/sdkman-init.sh && sdk install java $JAVA_VERSION && sdk use java $JAVA_VERSION'

# Install cursor-agent for the vscode user
USER $USER
RUN set -e && \
    echo "Installing Cursor Agent for user $USER..." && \
    curl -fsSL https://cursor.com/install | bash && \
    echo "Cursor Agent installation completed" && \
    # Verify installation
    ls -la /home/$USER/.local/bin/ || echo "Local bin directory contents:" && \
    sleep 2

# Add cursor-agent to PATH for the user
ENV PATH="/home/$USER/.local/bin:$PATH"

# Verify cursor-agent installation
RUN echo "Verifying cursor-agent installation..." && \
    which cursor-agent || echo "cursor-agent not found in PATH, checking alternatives..." && \
    find /home/$USER -name "*cursor*" -type f 2>/dev/null || echo "No cursor files found" && \
    # Check if cursor-agent is executable
    if command -v cursor-agent >/dev/null 2>&1; then \
        echo "cursor-agent found and executable"; \
        cursor-agent --version || echo "cursor-agent version check failed"; \
    else \
        echo "cursor-agent not found in PATH"; \
        echo "Current PATH: $PATH"; \
        ls -la /home/$USER/.local/bin/ || echo "Local bin directory not found"; \
    fi

# Create workspace directory for cursor-agent
RUN mkdir -p /home/$USER/workspace

# Create a helper script for running cursor-agent
RUN cat > /home/$USER/run-cursor-agent.sh << 'EOF'
#!/bin/bash
set -e

echo "=== Cursor Agent Helper Script ==="

# Check if API token is provided
if [ -z "$CURSOR_API_KEY" ]; then
    echo "Error: CURSOR_API_KEY environment variable is required"
    echo "Usage: CURSOR_API_KEY=your_key_here ./run-cursor-agent.sh \"Your prompt here\""
    exit 1
fi

# Check if prompt is provided
if [ -z "$1" ]; then
    echo "Error: Please provide a prompt as the first argument"
    echo "Usage: CURSOR_API_KEY=your_key_here ./run-cursor-agent.sh \"Your prompt here\""
    exit 1
fi

PROMPT="$1"
WORKSPACE_DIR="${CURSOR_WORKSPACE:-/home/vscode/workspace}"
MODEL="${CURSOR_MODEL:-}"

echo "API Token: ${CURSOR_API_KEY:0:10}..." # Show only first 10 chars for security
echo "Prompt: $PROMPT"
echo "Workspace: $WORKSPACE_DIR"

# Ensure workspace directory exists
mkdir -p "$WORKSPACE_DIR"
cd "$WORKSPACE_DIR"
echo "Working in directory: $(pwd)"

# Check if cursor-agent is available
if command -v cursor-agent >/dev/null 2>&1; then
    echo "Cursor Agent found at: $(which cursor-agent)"
    
    # Build command with optional model
    CMD="cursor-agent --api-key $CURSOR_API_KEY --print --force"
    if [ -n "$MODEL" ]; then
        CMD="$CMD --model $MODEL"
        echo "Using model: $MODEL"
    fi
    CMD="$CMD \"$PROMPT\""
    
    echo "Executing cursor-agent..."
    eval $CMD
else
    echo "Error: cursor-agent not found in PATH"
    echo "PATH: $PATH"
    exit 1
fi
EOF

# Make the helper script executable
RUN chmod +x /home/$USER/run-cursor-agent.sh && chown $USER:$USER /home/$USER/run-cursor-agent.sh

# Switch back to root for any remaining setup
USER root