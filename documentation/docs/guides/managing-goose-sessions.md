---
sidebar_position: 1
title: Managing Goose Sessions
sidebar_label: Managing Sessions
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


A session is a single, continuous interaction between you and Goose, providing a space to ask questions and prompt action. In this guide, we'll cover how to start, exit, and resume a session. 


## Start Session 

<Tabs>
    <TabItem value="ui" label="Goose Desktop" default>
        After choosing an LLM provider, you'll see the session interface ready for use. Type your questions, tasks, or instructions directly into the input field, and Goose will immediately get to work. 

        To start a new session at any time, click the three dots in the top-right corner of the application and select **New Session** from the dropdown menu.

    </TabItem>
    <TabItem value="cli" label="Goose CLI">
        From your terminal, navigate to the directory from which you'd like to start, and run:
        ```sh
        goose session 
        ```
    </TabItem>
</Tabs>

:::info
If this is your first session, Goose will prompt you for an API key to access an LLM (Large Language Model) of your choice. For more information on setting up your API key, see the [Installation Guide](/docs/getting-started/installation#set-llm-provider). Here is the list of [supported LLMs](/docs/getting-started/providers).
:::

## Name Session
<Tabs>
    <TabItem value="ui" label="Goose Desktop" default>
        Within the Desktop app, sessions are automatically named using the current timestamp in the format `YYYYMMDD_HHMMSS`. Goose also provides a description of the session based on context.
    </TabItem>
    <TabItem value="cli" label="Goose CLI">
        By default, Goose names your session using the current timestamp in the format `YYYYMMDD_HHMMSS`. If you'd like to provide a specific name, this is where you'd do so. For example to name your session `react-migration`, you would run:

        ```
        goose session -n react-migration
        ```

        You'll know your session has started when your terminal looks similar to the following:

        ```
        starting session | provider: openai model: gpt-4o
        logging to ~/.local/share/goose/sessions/react-migration.json1
        ```
    </TabItem>
</Tabs>

## Exit Session
Note that sessions are automatically saved when you exit.
<Tabs>
    <TabItem value="ui" label="Goose Desktop" default>
    To exit a session, simply close the application.
    </TabItem>    
    <TabItem value="cli" label="Goose CLI">
        To exit a session, type `exit`. Alternatively, you exit the session by holding down `Ctrl+C`.

        Your session will be stored locally in `~/.local/share/goose/sessions`.
    </TabItem>
</Tabs>

## Resume Session

<Tabs>
    <TabItem value="ui" label="Goose Desktop" default>
    1. Click `...` in the upper right corner
    2. Click `Previous Sessions`
    3. Click a session
    4. Click `Resume Session` in the upper right corner
    </TabItem>
    <TabItem value="cli" label="Goose CLI">
        To resume your latest session, you can run the following command:

        ```
         goose session -r
        ```

        To resume a specific session, run the following command: 

        ```
        goose session -r --name <name>
        ```
        For example, to resume the session named `react-migration`, you would run:

        ```
        goose session -r --name react-migration
        ```

        :::tip
        While you can resume sessions using the commands above, we recommend creating new sessions for new tasks to reduce the chance of [doom spiraling](/docs/troubleshooting#stuck-in-a-loop-or-unresponsive).
        :::

    </TabItem>
</Tabs>

### Search Session History

<Tabs>
    <TabItem value="ui" label="Goose Desktop" default>
        In Goose Desktop, you can search session metadata including the description, filename, and working directory path. The search is text-based and supports case-sensitive matching, but doesn't search session content or support regex patterns.

        1. Click `...` in the upper right corner
        2. Click `Previous Sessions`
        3. Use `Cmd+F` to open the search bar
        4. Enter your search term
        5. Use search features to refine and navigate results

        | Action | macOS | Windows/Linux |
        |--------|-------|---------------|
        | Next Match | `Cmd+G`<br />or `↓` | `Ctrl+G`<br />or `↓` |
        | Previous Match | `Shift+Cmd+G`<br />or `↑` | `Shift+Ctrl+G`<br />or `↑` |
        | Toggle Case-Sensitivity | `Aa` | `Aa` |
        | Focus Search Bar | `Cmd+F` | `Ctrl+F` |
        | Close Search | `Esc` or X | `Esc` or X |

    </TabItem>
    <TabItem value="cli" label="Goose CLI">
        The Goose CLI supports [listing session history](/docs/guides/goose-cli-commands/#session-list-options) but doesn't provide search functionality. As a workaround, you can use your terminal's search capabilities (including regex support). Examples for macOS:

        ```bash
        # Search session IDs (filenames)
        ls ~/.local/share/goose/sessions/ | grep "full or partial session id"

        # List sessions modified in last 7 days
        find ~/.local/share/goose/sessions/ -mtime -7 -name "*.jsonl"

        # Show first line (metadata) of each session file
        for f in ~/.local/share/goose/sessions/*.jsonl; do
            head -n1 "$f" | grep "your search term" && echo "Found in: $(basename "$f" .jsonl)"
        done

        # Find search term in session content
        rg "your search term" ~/.local/share/goose/sessions/

        # Search and show session IDs that contain search term
        for f in ~/.local/share/goose/sessions/*.jsonl; do
          if grep -q "your search term" "$f"; then
            echo "Found in session: $(basename "$f" .jsonl)"
          fi
        done
        ```

    </TabItem>
</Tabs>

### Resume Session Across Interfaces

You can resume a CLI session in Desktop and vice versa.

<Tabs>
    <TabItem value="ui" label="Goose Desktop" default>
    All saved sessions are listed in the Desktop app, even CLI sessions. To resume a CLI session within the Desktop:

    1. Click `...` in the upper right corner
    2. Click `Previous Sessions`
    3. Click the session you'd like to resume

    :::tip
    If you named the session, you'll recognize the filename. However, if you don't remember the exact session name, there is a description of the topic.
    :::

    4. Click `Resume Session` in the upper right corner

    :::note Example

    **CLI Command**

    ```sh
    goose session -n react-migration
    ```

    **Desktop Session**

    | Session Description     | Session Filename             |
    |-------------------------|------------------------------|
    | Code Migration to React | **react-migration**.jsonl    | 



    :::
    </TabItem>
    <TabItem value="cli" label="Goose CLI">
    To resume a Desktop session within CLI, get the name of the session from the Desktop app. Note that unless you specifically named the session, its default name is a timestamp in the format `YYYYMMDD_HHMMSS`.

    1. Open Goose Desktop
    2. Click `...` in the upper right corner
    3. Click `Previous Sessions`
    4. Find the session that you want to resume, and copy the basename (without the `.jsonl` extension). 
    :::note Example

    **Desktop Session**

    | Session Description    | Session Filename             |
    |------------------------|------------------------------|
    | GitHub PR Access Issue | **20250305_113223**.jsonl    | 


    **CLI Command**
    ```sh
    goose session -r --name 20250305_113223
    ```
    :::

    </TabItem>
</Tabs>

## Remove Sessions

You can remove sessions using CLI commands. For detailed instructions on session removal, see the [CLI Commands documentation](/docs/guides/goose-cli-commands#session-remove-options).

## Search Within Sessions

Search allows you to find specific content within your current session. The search functionality is available in both CLI and Desktop interfaces.

<Tabs>
    <TabItem value="ui" label="Goose Desktop" default>
        Trigger search using keyboard shortcuts or the search icon:

        | Action | macOS | Windows/Linux |
        |--------|-------|---------------|
        | Open Search | `Cmd+F`  | `Ctrl+F`  |
        | Previous Match | `↑` | `↑` |
        | Next Match | `↓` | `↓` |
        | Toggle Case-Sensitivity | `Aa` | `Aa` |
        | Close Search | `Esc` or X | `Esc` or X |

    </TabItem>
    <TabItem value="cli" label="Goose CLI">
        Search functionality is provided by your terminal interface. Use the appropriate shortcut for your environment:

        | Terminal | Operating System | Shortcut |
        |----------|-----------------|-----------|
        | iTerm2 | macOS | `Cmd+F` |
        | Terminal.app | macOS | `Cmd+F` |
        | Windows Terminal | Windows | `Ctrl+F` |
        | Linux Terminal | Linux | `Ctrl+F` |

        :::info
        Your specific terminal emulator may use a different keyboard shortcut. Check your terminal's documentation or settings for the search command.
        :::
    </TabItem>
</Tabs>