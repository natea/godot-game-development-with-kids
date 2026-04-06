# Godot Game Development with Kids

An AI-assisted approach to teaching kids game development using the Godot Engine and agentic coding tools.

This project is based on [Maaack's Godot Game Template](https://github.com/Maaack/Godot-Game-Template), which provides a solid foundation with main menus, options, pause menus, credits, scene loaders, and an example game scene. See the [original template documentation](/addons/maaacks_game_template/docs/) for details on those features.

## Getting Started with Agentic Game Development

The idea is simple: your kid talks to an AI coding assistant in natural language, and the AI writes the game code while the Godot preview updates in real time.

### Recommended Setup

For the best experience introducing kids to agentic game development:

1. **Use [Cursor](https://cursor.sh/) or [Windsurf](https://codeium.com/windsurf)** as your code editor. These have a prominent chat window that will be the main way your kid interacts with the coding agent.

2. **Split the screen 50/50** -- one half for the coding assistant's chat window, the other half for the Godot editor (especially the game preview). As your kid types commands to the AI, the Godot preview window updates to show the changes.

3. **Keep the Godot game preview visible** so kids get immediate visual feedback on what the AI is building for them.

## MCP Server for Godot (GDAI MCP)

This project uses the [MCP Server for Godot Game Engine (GDAI MCP)](https://gdaimcp.com/), which gives your AI coding assistant direct access to Godot's scene tree, nodes, and game state.

**GDAI MCP is a commercial product and is not included in this repository.** To set it up:

1. Purchase and download GDAI MCP from [gdaimcp.com](https://gdaimcp.com/)
2. Place it in the `addons/` directory of this project
3. Open the project in Godot and enable the addon via **Project > Project Settings > Plugins**
4. Configure your coding assistant to use the MCP server (see below)

<img width="2806" height="1966" alt="image" src="https://github.com/user-attachments/assets/64bdbc9a-1976-43ff-a6e5-ad3fde4620ae" />

### Adding the MCP Server to Your Coding Assistant

#### Claude Code

Add the MCP server to your project settings in `.claude/settings.json`:

```json
{
  "mcpServers": {
    "godot": {
      "command": "path/to/gdai-mcp-server",
      "args": []
    }
  }
}
```

Or add it globally in `~/.claude/settings.json` to use across all projects.

#### Cursor

Go to **Settings > MCP** and add a new server with the command and arguments for the GDAI MCP server.

#### Windsurf

Configure MCP servers in Windsurf's settings under the MCP section, pointing to the GDAI MCP server executable.

#### Other Assistants (Codex, Gemini, OpenCode, etc.)

Most coding assistants that support MCP follow a similar pattern -- you provide the server command and arguments in the assistant's configuration file. Check your assistant's documentation for the specific MCP server configuration format.

## Godot Development Skill

This project includes a Godot development skill at `.claude/skills/godot/SKILL.md` that provides specialized knowledge of Godot's file formats (`.gd`, `.tscn`, `.tres`), architecture patterns, validation tools, code templates, and CLI workflows.

The skill is designed for Claude Code but can be used with other AI coding assistants:

- **Codex, Gemini, OpenCode, etc.** -- Copy the `.claude/skills/godot/` directory into your coding assistant's equivalent skills directory.

This skill originated from the [terma project by Ben Follington](https://github.com/bfollington/terma/blob/4a88bbd3f0ae3ecb598023f29b4e848c1ec9113b/plugins/tsal/skills/godot/SKILL.md).

## Template Features

This project inherits all features from [Maaack's Godot Game Template](https://github.com/Maaack/Godot-Game-Template):

- Main Menu, Options Menus, Pause Menu, Credits
- Loading Screen, Opening Scene
- Persistent Settings, Simple Config Interface
- Keyboard/Mouse and Gamepad Support
- UI Sound Controller, Background Music Controller
- Level Loaders, Level Progress Manager
- Win / Lose Manager
- Example Game Scene with Levels and Tutorials

See the [template documentation](/addons/maaacks_game_template/docs/) for full details.

## Links

- [Maaack's Godot Game Template](https://github.com/Maaack/Godot-Game-Template) -- the base template this project is built on
- [GDAI MCP](https://gdaimcp.com/) -- MCP Server for Godot Game Engine
- [Godot Engine](https://godotengine.org/) -- the open-source game engine
- [Terma Godot Skill](https://github.com/bfollington/terma/blob/4a88bbd3f0ae3ecb598023f29b4e848c1ec9113b/plugins/tsal/skills/godot/SKILL.md) -- origin of the included Godot skill

## Attribution

- Game template by [Maaack](https://github.com/Maaack/Godot-Game-Template) ([License](/addons/maaacks_game_template/LICENSE.txt))
- Godot skill adapted from [terma](https://github.com/bfollington/terma) by Ben Follington
