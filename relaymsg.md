# `overdrivenetworks.com/relaymsg` extension

The `relaymsg` extension allows bots to send channel messages using spoofed nicks. The goal is to allow creating a transparent experience for relay bots, without the overhead of connecting extra clients or maintaining state as a virtual server.

## The `overdrivenetworks.com/relaymsg` capability

The `overdrivenetworks.com/relaymsg` capability lets clients use the `RELAYMSG` command and receive `relaymsg` tags.

If this capability has a value, the given characters are 'nickname separators'. These characters aren't allowed in normal nicknames, and if given one MUST be present in spoofed nicknames. For example, with `overdrivenetworks.com/relaymsg=/` the spoofed nickname MUST include the character `"/"`.

## `RELAYMSG` command

The `RELAYMSG` command takes the following arguments:

```
RELAYMSG <channel> <spoofed nick> :<message>
```

Upon receiving this command, the IRCd will translate the message to a `PRIVMSG`:

```
@relaymsg=<botnick> :spoofednick!<ident>@<host> PRIVMSG <channel> :<message>
```

Clients MUST request the `overdrivenetworks.com/relaymsg` capability before using this command. The capability also lets clients receive the `relaymsg` message tag, which is set to the nick of the sender. This allows the sender to distinguish relayed messages from those sent by other clients, preventing forwarding loops in the case of relay bots.

The `ident` and `host` fields are defined by the server implementation and may be configurable.

## Abuse prevention

In order to prevent abuse, servers should take steps to verify that the caller is authorized to use `RELAYMSG`, and that the spoofed nick cannot be used to confuse users. This may include:

- Restricting `RELAYMSG` to IRC operators, channel operators, or other approved users
- Defining a nickname separator (described above) and requiring that spoofed nicks contain one
- Checking that the spoofed nick is not in use
- Checking that the sender is in the target channel

Servers MUST also sanitize or reject nicks that contain reserved IRC characters, including `!+%@&#$:'"?*,.` and whitespace. This is to avoid sending invalid messages to clients.

Servers may choose to filter spoofed nicks further, or pass them through as is. Spoofed nicks do not necessarily need to be valid IRC nicks; implementations may choose to accept UTF-8 text, for example. One benefit to having spoofed nicks always be invalid is preventing IRC users from changing their nick to one of those that the bot is using.

## Example implementations

### Server side

- InspIRCd 3.x via custom module: https://github.com/overdrivenetworks/inspircd-contrib/blob/relaymsg/3.0/m_relaymsg.cpp

### Client side

- matterbridge fork: https://github.com/overdrivenetworks/matterbridge/blob/overdrive/bridge/irc/irc.go#L198-L246
