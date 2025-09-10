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