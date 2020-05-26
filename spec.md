# `overdrivenetworks.com/relaymsg` extension

The `relaymsg` extension allows bots to send channel messages using spoofed nicks. The goal is to allow creating a transparent experience for relay bots, without the overhead of connecting extra clients or maintaining state as a virtual server.

## `RELAYMSG` command

The `RELAYMSG` command takes the following arguments:

```
RELAYMSG <channel> <spoofed nick> :<message>
```

Upon receiving this command, the IRCd will translate the message to a `PRIVMSG`:

```
@relaymsg=<botnick> :spoofednick!<ident>@<host> PRIVMSG <channel> :<message>
```

In order to use this command, clients MUST request the `overdrivenetworks.com/relaymsg` capability beforehand. This provides the `relaymsg` message tag, which is set to the nick of the caller. In turn, this allows the sender to distinguish between messages it sent and those from other clients, preventing forwarding loops in the case of relay bots.

The `ident` and `host` fields are defined by the server implementation and may be configurable.

## Abuse prevention & Spoofed nick validity

In order to prevent abuse, servers should take steps to verify that the caller is authorized to use `RELAYMSG`, and that the spoofed nick cannot be used to confuse users. This may include:

- Restricting `RELAYMSG` to IRC operators or a certain set of hosts
- Restricting spoofed nicks to a certain prefix, suffix, or nick glob (e.g. `*/*`)
- Checking that the spoofed nick is not in use
- Checking that the sender is in the target channel

Servers MUST also sanitize or reject nicks that contain reserved IRC characters, including `!+%@&#$:'"?*,.` and whitespace. This is to avoid sending invalid messages to clients.

Servers may choose to filter spoofed nicks further, or pass them through as is. Spoofed nicks do not necessarily need to be valid IRC nicks; implementations may choose to accept UTF-8 text, for example. One beneficial side effect of having spoofed nicks always be invalid (e.g. by requiring a separator like `/`) is preventing IRC users from changing their nick to those that the bot is using.

## Example implementations

### Server side

- InspIRCd 3.x via custom module: https://github.com/overdrivenetworks/inspircd-contrib/blob/relaymsg/3.0/m_relaymsg.cpp

### Client side

- matterbridge fork: https://github.com/overdrivenetworks/matterbridge/blob/overdrive/bridge/irc/irc.go#L198-L246
