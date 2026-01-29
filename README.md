# Android-Skills

Welcome to **Android-Skills**! 🤖✨

This is a community-driven repository dedicated to sharing and curating "skills" for AI agents. Whether you are building autonomous agents, chatbots, or workflow automations, this collection aims to provide modular, reusable skill definitions that can be easily integrated into your AI systems.

## 📂 Repository Structure

We maintain a strict yet simple folder structure to ensure consistency and ease of use:

```
android-skills/
├── README.md           # This documentation
├── template/           # A template to get you started
│   └── SKILL.md
├── <skill-name>/       # Your awesome skill directory
│   ├── SKILL.md        # The definition file for the skill
│   └── AGENTS.md       # Optional: Agent usage guide (see below)
└── ...
```

- **`<skill-name>/`**: Each skill gets its own dedicated directory. The name should be kebab-case (e.g., `weather-checker`, `git-commit-helper`).
- **`SKILL.md`**: The core file containing the skill's metadata, instructions, patterns, and examples.
- **`AGENTS.md`** (optional): A companion file that provides agent-specific guidance on how to use the skill effectively. Include this when the skill requires specific context for AI agents.

## 🚀 How to Use

Browse the directories to find a skill that fits your agent's needs. You can:

1.  **Directly Copy**: Copy the content of the `SKILL.md` into your agent's system prompt or tool definition.
2.  **Reference**: If your agent supports loading skills from files, point it to the specific `SKILL.md` path.
3.  **Adapt**: Use these skills as a base and modify them to fit your specific context.

## 🤝 Contributing

We love contributions! If you have a useful prompt or agent skill, please share it with the community.

### Step-by-Step Guide

1.  **Fork** this repository.
2.  **Create a Branch** for your new skill: `git checkout -b add-my-new-skill`.
3.  **Create a Directory** using kebab-case for your skill name: `mkdir my-new-skill`.
4.  **Add `SKILL.md`**: Create the file inside your directory. You can copy the `template/SKILL.md` to get started.
    - Ensure you include a clear **Description**.
    - Provide detailed **Instructions** for the AI.
    - Include **Examples** of inputs and outputs.
5.  **Add `AGENTS.md`** (optional): If your skill benefits from agent-specific guidance, create an `AGENTS.md` with:
    - Quick start for agents
    - Common tasks with examples
    - Key patterns and tips
6.  **Commit and Push** your changes.
7.  **Open a Pull Request** describing what your skill does.

### Guidelines for a Good Skill

- **Clarity**: Be specific in your instructions to the AI.
- **Modularity**: Keep skills focused on a single domain or task.
- **Examples**: Few-shot examples significantly improve performance.
- **AGENTS.md**: Consider adding when the skill has complex usage patterns or requires specific context.

## 📄 License

This project is open source and available to everyone. Please feel free to use, modify, and distribute these skills.