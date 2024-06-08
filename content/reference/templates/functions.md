+++
title = 'Functions'
weight = 1
+++

Functions are used to take action within template scripts. Some functions accept arguments, and some functions return
values you can send in your response or use as arguments for other functions.

<!--more-->

> Functions are underappreciated. In general, not just in templates. // Rob Pike

{{% notice info %}}

Every function having both cases possible for an argument - ID/name, then this name is handled case insensitive, for
example `getRole "yagpdb"` and `getRole "yAgPdB"` would have same responses even if server has both of these roles, so
using IDs is better.

{{% /notice %}}

### Channel

{{% notice info %}}

The ratelimit for editing a channel is 2 requests per 10 minutes per channel.

{{% /notice %}}

|**Function**| **Description**|
|-| -|
|`addThreadMember` thread member| Adds a member to an existing thread. Does nothing if either thread or member arguments are invalid. Argument `thread` can be either ID, "name" or even `nil` for current thread, and `member` can either be ID or @ mention.|
|`createForumPost` channel name content (values)| Creates a new forum post in forum `channel`. Argument `name` represents the title of the post, `content` can be a string, an embed or a complex message. The optional `"values"` argument  consists of key-value pairs:`"slowmode" value` allows the user to specify the thread's slowmode in seconds. `"tags" value` allows the user to specify one or more forum tags. Duplicate and invalid tags are ignored, and its `value` can be either a string or cslice for multiple tags. Returns *templates.CtxChannel upon success.<br><br>Example: `{{createForumPost &#x3C;forumChannelID> "Post Title" "content as a string" "slowmode" 60}}`.|
|`createThread` channel messageID name (values)| Creates a new thread in `channel`. `messageID` can point to a valid message (creates a message thread) or nil. `name` is the title of the thread. Optional `values` determine how thread is created and come in following order - `private` of type _bool_ determines whether the thread is private;  `auto_archive_duration` must be an integer between 60 and 10080 and it determines when the thread will stop showing in the channel list after given minutes of inactivity, can be set to: 60, 1440, 4320, 10080 ; `invitable` of type _bool_, whether non-moderators can add other non-moderators to a thread, it is only available on private threads. Returns *templates.CtxChannel upon success.<br>Example: `{{createThread nil nil "YAGPDB is cool" true 60 true}}` will create a thread named "YAGPDB is cool" under current channel without any message reference, thread is private, it will get archived in an hour and non-moderators can add others to thread.|
|`deleteForumPost` post| Deletes existing post in the forum - `post` can be either its ID, post’s name or even `nil` if  done inside that post. If supplied argument is not a valid active post then it is ignored. Is an alias of `deleteThread` function.|
|`deleteThread` thread| Deletes existing thread - `thread` can be either its ID, thread’s name or even `nil` if  done inside that thread. If supplied argument is not a valid active thread then it is ignored.|
|`editChannelName` <br>channel "newName"| Function edits channel's name. `channel` can be either ID, "name" or even `nil` if triggered in that channel name change is intended to happen. `"newName"` has to be of type _string_. For example  >`{{editChannelName nil (print "YAG - " (randInt 1000))}}`|
|`editChannelTopic` channel "newTopic"| Function edits channel's topic/description. `channel` can be either ID, "name" or `nil` if triggered in that channel where name change is intended to happen.  `"newTopic"` has to be of type _string_. For example >`{{editChannelTopic nil "YAG is cool"}}`|
|`getChannel` channel| Function returns full channel object of given `channel` argument which can be either its ID, name or `nil` for triggering channel, and is of type _\*templates.CtxChannel_. For example > `{{(getChannel nil).Name}}` returns the name of the channel command was triggered in.|
|`getChannelOrThread` channel| Returns type _\*templates.CtxChannel_ corresponding to [Channel](/reference/templates#channel) object.|
|`getChannelPins` channel| Returns a slice of all pinned message objects in targeted channel.|
|`getPinCount` channel| Returns the count of pinned messages in given channel which can be either its ID, name or `nil` for triggering channel. Can be called 2 times for regular and 4 for premium servers.|
|`getThread` channel| Returns type _\*templates.CtxChannel_ corresponding to [Channel](/reference/templates#channel) object.|
|`removeThreadMember` thread member| Removes a member from an existing thread. Does nothing if either thread or user are invalid. Argument `thread` can be either ID, "name" or even `nil` for current thread, and `member` can either be ID or @ mention.|

### Database

|**Function**| **Description**|
|-| -|
|`dbBottomEntries` pattern amount nSkip| Returns `amount (max 100)`top entries of keys determined by the `pattern` from the database, sorted by the numeric value in a ascending order, next by entry ID.|
|`dbCount` (userID\|pattern\|query)| Returns the count of all database entries which are not expired. Optional arguments: if `userID` is given, counts entries for that userID; if `pattern`, only those keys are counted that match the given pattern; and if `query` is provided, it should be an sdict with the following keys:<ul><li>`userID` - only counts entries with that userID, defaults to counting entries with any userID.</li><li>`pattern` - only counts dbEntry keys with names matching the pattern given, defaults to counting entries with any name.</li></ul>|
|`dbDel` userID key| Deletes the specified key for the specified value from the database.|
|`dbDelByID` userID ID| Deletes database entry by its ID.|
|`dbDelMultiple` query amount skip| Deletes `amount (max 100)` entries from the database matching the criteria provided. `query` should be an _sdict_ with the following options:<ul><li>`userID` - only deletes entries with the dbEntry field .UserID provided, defaults to deleting entries with any ID.</li><li>`pattern` - only deletes entry keys with a name matching the pattern given.</li><li>`reverse` - if true, starts deleting entries with the lowest values first; otherwise starts deleting entries with the highest values first. Default is `false`.</li></ul>Returns the number of rows that got deleted or an error.|
|`dbGet` userID key| Retrieves a value from the database for the specified user, this returns DBEntry object. Does not fetch member data as user object for .User like `dbGetPattern`, `dbBottom/TopEntries` do.|
|`dbGetPattern` userID pattern amount nSkip| Retrieves up to`amount (max 100)`entries from the database in ascending order.|
|`dbGetPatternReverse` userID pattern amount nSkip| Retrieves`amount (max 100)`entries from the database in descending order.|
|`dbIncr` userID key incrBy| Increments the value for specified key for the specified user, if there was no value then it will be set to `incrBy`. Also returns the entry's current, increased value.|
|`dbRank` query userID key| Returns the rank of the entry specified by the user ID and key provided in the set of entries matching the criteria provided. `query` specifies the set of entries that should be considered, and should be a sdict with the following options:<ul><li>`userID` - only includes entries with that user ID, defaults to including entries with any user ID</li><li>`pattern` - only includes database's `key` entries with names matching the pattern given, defaults to counting entries with any name</li><li>`reverse` - if true, entries with lower value have higher rank; otherwise entries with higher value have higher rank. Default is `false`.</li></ul>|
|`dbSet` userID key value| Sets the value for the specified `key` for the specific `userID` to the specified `value`. `userID` can be any number of type _int64_.  <br><br>Values are stored either as of type _float64_ (for numbers, oct or hex) or as varying type in bytes (for _slices_, _maps_, _strings_ etc) depending on input argument.|
|`dbSetExpire` userID key value ttl| Same as `dbSet` but with an expiration `ttl` which is an _int_ and represents seconds.|
|`dbTopEntries` pattern amount nSkip| Returns `amount (max 100)`top entries of keys determined by the `pattern` from the database, sorted by the numeric value in a descending order, next by entry ID.|

Patterns are basic PostgreSQL patterns, not Regexp: An underscore `(_)`  matches any single character; a percent sign
`(%)` matches any sequence of zero or more characters.

{{% notice info %}}

**Note about saving numbers into database:** As stated above, database stores numbers as type _float64_. If you save a
large number into database like an _int64_ (which IDs are), the value will be truncated. To avoid this behavior, you can
convert the number to type _string_ before saving and later back to its original type when retrieving it. Example: `{{$v
:= .User.ID}} {{dbSet 0 "userid" (str $v)}} {{$fromDB := toInt (dbGet 0 "user_id").Value}}`

`dict` key values are also retrieved as _int64,_ so to use them for indexing one has to e.g. `index $x (toInt64 0)`

{{% /notice %}}

### ExecCC

{{% notice warning %}}

`execCC` calls are limited to 1 / CC for non-premium users and 10 / CC for premium users.

{{% /notice %}}

|**Function**| **Description**|
|-| -|
|`cancelScheduledUniqueCC` ccID key| Cancels a previously scheduled custom command execution using `scheduleUniqueCC`.|
|`execCC` ccID channel delay data| Function that executes another custom command specified by `ccID`. With delay 0 the max recursion depth is 2 (using `.StackDepth` shows the current depth). `execCC` is rate-limited strictly at max 10 delayed custom commands executed per channel per minute, if you go over that it will be simply thrown away. Argument `channel` can be `nil`, channel's ID or name. The`delay` argument is execution delay of another CC is in seconds. The `data` argument is content that you pass to the other executed custom command. To retrieve that `data` you use `.ExecData`. This example is important > [execCC example](/reference/custom-command-examples#countdown-example-exec-cc) also next snippet which shows you same thing run using the same custom command > [Snippets](#execcc-sections-snippets).  `execCC` is also thoroughly covered in this [GitHub gist](https://gist.github.com/l-zeuch/9f10d128184509ad531778f26550ed6d).|
|`scheduleUniqueCC` ccID channel delay key data| Same as `execCC`except there can only be 1 scheduled cc execution per server per key (unique name for the scheduler), if key already exists then it is overwritten with the new data and delay (as above, in seconds).An example would be a mute command that schedules the unmute action sometime in the future. However, let's say you use the unmute command again on the same user, you would want to override the last scheduled unmute to the new one. This can be used for that.|

#### ExecCC section's snippets

* To demonstrate execCC  and .ExecData using the same CC.

```go
{{ $yag := "YAGPDB rules! " }}
{{ $ctr := 0 }} {{ $yourCCID := .CCID }}
{{ if .ExecData }}
    {{ $ctr = add .ExecData.number 1 }}
    {{ $yag = print $yag $ctr }} {{ .ExecData.YAGPDB }}
{{ else }}
    So, someone rules.
    {{ $ctr = add $ctr 1 }} {{ $yag = print $yag 1 }}
{{ end }}
{{ if lt $ctr 5 }}
    {{ execCC $yourCCID nil 3 (sdict "YAGPDB" $yag "number" $ctr) }}
{{ else }} FUN'S OVER! {{ end }}
```

### Math

{{% notice info %}}

Boolean logic (and, not, or) and comparison operators (eq, gt, lt, etc.) are covered in [conditional
branching](/reference/templates#if-conditional-branching).

{{% /notice %}}

|**Function**| **Description**|
|-| -|
|`add` x y z ...| Returns x + y + z + ...,  detects first number's type - is it _int_ or _float_ and based on that adds. (use `toFloat` on the first argument to force floating point math.)`{{add 5 4 3 2 -1}}` sums all these numbers and returns `13`.|
|`bitwiseAnd` x y| The output of bitwise AND is 1 if the corresponding bits of two operands is 1. If either bit of an operand is 0, the result of corresponding bit is evaluated to 0. Example: `{{bitwiseAnd 12 25}}` returns `8`, that in binary 00001100 AND 00011001 is 00001000.|
|`bitwiseAndNot` x y| This function is called bit clear because of AND NOT. For example in the expression z = x AND NOT y, each bit of z is 0 if the corresponding bit of y is 1; otherwise it equals to the corresponding bit of x. `{{bitwiseAndNot 7 12}}` returns `3`, that is 0111 AND NOT 1100 is 11.|
|`bitwiseNot` x| The bitwise NOT operator inverts the bits of the argument. Example: `{{bitwiseNot 7}}` returns `-8`. that in binary 0111 to 1000|
|`bitwiseOr` x y z...| The output of bitwise OR is 1 if at least one corresponding bit of two operands is 1. Example: `{{bitwiseOr 12 25}}` returns `29`, that in binary 00001100 OR 00011001 is 00011101.|
|`bitwiseXor` x y| The result of bitwise XOR operator is 1 if the corresponding bits of two operands are opposite. Example: `{{bitwiseXor 12 25}}` returns `21`, that in binary 00001100 OR 00011001 is 00010101.|
|`bitwiseLeftShift` x y| Left shift operator shifts all bits towards left by a certain number of specified bits. The bit positions that have been vacated by the left shift operator are filled with 0. Example: `{{range seq 0 3}} {{bitwiseLeftShift 212 .}} {{end}}` returns `212 424 848`|
|`bitwiseRightShift` x y| Right shift operator shifts all bits towards right by certain number of specified bits. Example: `{{range seq 0 3}} {{bitwiseRightShift 212 .}} {{end}}` returns `212 106 53`.|
|`cbrt` x| Returns the cube root of given argument in type _float64_ e.g. `{{cbrt 64}}` returns `4`.|
|`div` x y z ...| Division, like `add` or `mult`, detects first number's type first. `{{div 11 3}}` returns `3` whereas `{{div 11.1 3}}` returns  `3.6999999999999997`|
|`fdiv` x y z ...| Meant specifically for floating point numbers division.|
|`log` x (base)| Log is a logarithm function using (log base of x). Arguments can be any type of numbers, as long as they follow logarithm logic. Return value is of type _float64_. If base argument is not given It is using natural logarithm (base e - The Euler's constant) as default.`{{ log "123" 2 }}` will return `6.94251450533924`.|
|`mathConst` "arg"| Function returns all constants available in golang's math package as _float64_. `"arg"` has to be a case-insensitive _string_ from [math constants list](https://pkg.go.dev/math@go1.18.2#pkg-constants). For example `{{mathConst "sqrtphi"}}` would return `1.272019649514069`.|
|`max` x y| Returns the larger of x or y as type _float64_.|
|`min` x y| Returns the smaller of x or y as type _float64_.|
|`mod` x y| Mod returns the floating-point remainder of the division of x by y.  For example, `mod 17 3` returns `2` as type _float64_.<br><br>Note that like Go's `[math.Mod](https://pkg.go.dev/math#Mod)` function, `mod` takes the sign of `x`, so `mod -5 3` results in `-2`, not `1`. To ensure a non-negative result, use `mod` twice, like such: `mod (add (mod x y) y) y`.|
|`mult` x y z ...| Multiplication, like `add` or `div`, detects first number's type. `{{mult 3.14 2}}` returns `6.28`|
|`pow` x y| Pow returns x\*\*y, the base-x exponential of y which have to be both numbers. Type is returned as _float64_. `{{ pow 2 3 }}` returns `8`.|
|`randInt` (stop, or start stop)| Returns a random integer between 0 and stop, or start - stop if two args are provided.Result will be `start &#x3C;= random number &#x3C; stop`. Without arguments, range is 0..10. Example in section's [Snippets](#math-sections-snippets).|
|`round` x| Returns the nearest integer, rounding half away from zero. Regular rounding > 10.4 is `10` and 10.5 is `11`. All round functions return type _float64_, so use conversion functions to get integers. For more complex rounding, example in section's [Snippets](#math-sections-snippets).|
|`roundCeil` x| Returns the least integer value greater than or equal to input or rounds up.  `{{roundCeil 1.1}}` returns `2`.|
|`roundEven` x| Returns the nearest integer, rounding ties to even.<br>`{{roundEven 10.5}}` returns `10 {{roundEven 11.5}}` returns `12`.|
|`roundFloor` x| Returns the greatest integer value less than or equal to input or rounds down. `{{roundFloor 1.9}}` returns `1`.|
|`sqrt` x| Returns the square root of a number as type _float64_.<br>`{{sqrt 49}}` returns `7, {{sqrt 12.34 \|printf "%.4f"}} returns 3.5128`|
|`sub` x y z ...| Returns x - y -z - ... Works like add, just subtracts.|

#### Math section's snippets

* `{{$d := randInt 10}}` Stores random _int_ into variable `$d` (a random number from 0-9).
* To demonstrate rounding float to 2 decimal places.\
  `{{div (round (mult 12.3456 100)) 100}}` returns 12.35\
  `{{div (roundFloor (mult  12.3456 100)) 100}}` returns 12.34

### Member

|**Function**| **Description**|
|-| -|
|`getTargetPermissionsIn` memberID channelID| Returns target’s permissions in the given channel.|
|`editNickname` "newNick"| Edits triggering user's nickname, argument has to be of type _string_. YAGPDB's highest role has to be above the highest role of the member and bot can't edit owner's nickname.|
|`hasPermissions` arg| Returns true/false on whether triggering user has the permission bit _int64_ that is also set in .Permissions_._|
|`getMember` mention/userID| Function returns [Member object](/reference/templates/#member) having above methods. `{{(getMember .User.ID).JoinedAt}}` <br>is the same as `{{.Member.JoinedAt}}`|
|`onlineCount`| Returns the count of online users/members on current server.|
|`targetHasPermissions` memberID arg| Returns true/false on whether targeted member has the permission bit _int64_.|

Permissions are covered
[here](https://discord.com/developers/docs/topics/permissions#permissions-bitwise-permission-flags). For example to get
permission bit for "use application commands" `{{bitwiseLeftShift 1 32}}` would return _int64_ `4294967296`.

### Mentions

|**Function**| **Description**|
|-| -|
|`mentionEveryone`| Mentions `@everyone`.|
|`mentionHere`| Mentions `@here`.|
|`mentionRoleID` roleID| Mentions the role found with the provided ID.|
|`mentionRoleName` "rolename"| Mentions the first role found with the provided name (case-insensitive).|

There is also .Mention method available for channel, role, user structs/objects.

#### Mentions section's snippets

* `<@{{.User.ID}}>` Outputs a mention to the user that called the command and is the same as `{{.User.Mention}}`
* `<@###########>` Mentions the user that has the ID ###### (See [How to get IDs](/reference/how-to-get-ids) to get ID).
* `<#&&&&&&&&&&&>` Mentions the channel that has ID &&&&&& (See [How to get IDs](/reference/how-to-get-ids) to get ID).
* `<@&##########>` Mentions the role with ID ######## ([listroles](/commands/all-commands#listroles) command
  gives roleIDs). This is usable for example with `{{sendMessageNoEscape nil "Welcome to role <@&11111111...>"}}`.
  Mentioning that role has to be enabled server- side in Discord.
* `</cmdName:cmdID>` Mentions a slash command, and makes it clickable and interactive with proper arguments e.g.
  `</howlongtobeat:842397645104087061>`.

### Message

|**Function**| **Description**|
|-| -|
|`addMessageReactions` channel messageID emojis...| Same as `addReactions` or `addResponseReactions`, but can be used on any messages using its ID. `channel` can be either `nil`, channel's ID or its name.Emojis can be presented as a _cslice_. Example in section's [Snippets](#message-sections-snippets).|
|`addReactions` "👍" "👎" ...| Adds each emoji as a reaction to the message that triggered the command (recognizes Unicode emojis and `emojiName:emojiID`). Emojis can be presented as a _cslice_.|
|`addResponseReactions` "👍" "👎" ...| Adds each emoji as a reaction to the response message (recognizes Unicode emojis and `emojiName:emojiID`). Emojis can be presented as a _cslice_.|
|`complexMessage` "allowed\_mentions" arg "content" arg "embed" arg "file" arg "filename" arg "reply" arg "silent" arg| `complexMessage` creates a _so-called_ bundle of different message fields for `sendMessage...` functions to send them out all together. Its arguments need to be preceded by predefined keys:<br>`"allowed_mentions"` sends out very specific mentions where `arg` is an _sdict_ having keys: `"parse"` that takes a _cslice_ with accepted values for a type to mention - "users", "roles", "everyone"; _sdict_ keys`"users"`, `"roles"` take a _cslice_ of IDs either for users or roles and `"replied_user"` is a _bool_ true/false for mentioning replied user.`"content"` for regular text; `"embed"` for embed arguments created by `cembed` or `sdict`. With `cslice` you can use up to 10 embeds as arguments for `"embed"`. `"file"` for printing out content as a file with default name `attachment_YYYY-MM-DD_HH-MM-SS.txt` (max size 100 000 characters ca 100kB). `"filename"` lets you define custom file name if `"file"` is used with max length of 64 characters, extension name remains `txt`. `"reply"`replies to a message, where `arg`is messageID. If replied message is in another channel, `sendMessage`channel must be also that channel. To "reply-ping", use `sendMessageNoEscape`.`"silent"` suppresses message's push and desktop notifications, `arg` is _bool_ true/false.Example in this section's [Snippets](#message-sections-snippets).|
|`complexMessageEdit` "allowed\_mentions" arg "content" arg "embed" arg "silent" arg| Special case for `editMessage` function - either if `complexMessage` is involved or works even with regular message. Has parameters "allowed\_mentions",`"content", "embed"` and `"silent"` that edit the message and work the same way as for `complexMessage`. Example in this section's [Snippets](#message-sections-snippets).|
|`deleteAllMessageReactions` channel messageID (emojis...)| Deletes all reactions pointed message has. `channel` can be ID, "name" or `nil`. `emojis` argument is optional and works like it's described for the function `deleteMessageReaction`.|
|`deleteMessage` channel messageID (delay)| Deletes message with given `messageID` from `channel`. Channel can be either `nil`, channel's ID or its name. `(delay)` is optional and like following two delete functions, it defaults to 10 seconds, max being 1 day or 86400 seconds. Example in section's [Snippets](functions#message-sections-snippets).|
|`deleteMessageReaction` channel messageID userID emojis...| Deletes reaction(s) from a message. `channel` can be ID, "name" or `nil`. <br>`emojis` argument can be up to 10 emojis, syntax is `emojiName` for Unicode/Discord's default emojis and `emojiName:emojiID` for custom emotes.  <br>Example: `{{deleteMessageReaction nil (index .Args 1) .User.ID "👍" "👎"}}` will delete current user's reactions with thumbsUp/Down emotes from current running channel's message which ID is given to command as first argument `(index .Args 1)`.<br>Also usable with [Reaction trigger](/reference/templates#reaction).|
|`deleteResponse` (delay)| Deletes the response after a certain time from optional `delay` argument (max 86400 seconds = 1 day). Defaults to 10 seconds.|
|`deleteTrigger` (delay)| Deletes the trigger after a certain time from optional `delay` argument  (max 86400 seconds = 1 day). Defaults to 10 seconds.|
|`editMessage` channel messageID newMessageContent| Edits the message in channel, channel can be either `nil`, channel's ID or "name". Light example in section's [Snippets](#message-sections-snippets).|
|`editMessageNoEscape` channel messageID newMessageContent| Edits the message in channel and has same logic in escaping characters as `sendMessageNoEscape`.|
|`getMessage` channel messageID| Returns a [Message](/reference/templates#message) object. `channel` can be either its ID, name or nil for triggering channel.|
|`pinMessage` channel messageID| Pins a message by its ID in given channel. `channel` can be either its ID, name or nil for triggering channel. Can be called 5 times.|
|`publishMessage` channel messageID| Publishes a message by its ID in given announcement channel. `channel` can be either its ID, name or nil for triggering channel. Can be called once. This function will not work for leave or join messages.|
|`publishResponse`| Publishes the response when executed in an announcement channel. This function will not work for leave or join messages.|
|`sendDM` "message here"| Sends the user a direct message, only one DM can be sent per custom command (accepts embed objects). YAG will only DM triggering user. This function will not work for leave messages.|
|`sendMessage` channel message| Sends `message (string or embed)` in `channel`, channel can be either `nil`, the channel ID or the channel's "name".|
|`sendMessageNoEscape` channel message| Sends `message (string or embed)` in `channel`, channel can be either `nil`, the channel ID or the channel "name". Doesn't escape mentions (e.g. role mentions, reply mentions or @here/@everyone).|
|`sendMessageNoEscapeRetID` channel message| Same as `sendMessageNoEscape`, but also returns messageID to assigned variable for later use.|
|`sendMessageRetID` channel message| Same as `sendMessage`, but also returns messageID to assigned variable for later use. Example in section's [Snippets](#message-sections-snippets).|
|`unpinMessage` channel messageID| Unpins the message by its ID in given channel. `channel` can be either its ID, name or nil for triggering channel. Can be called 5 times.|

#### Message section's snippets

* Sends message to current channel `nil` and gets messageID to variable `$x`. Also adds reactions to this message. After
  5 seconds, deletes that message. >

    `{{$x := sendMessageRetID nil "Hello there!"}} {{addMessageReactions nil $x (cslice "👍" "👎") "`❤️`" }}
    {{deleteMessage nil $x 5}}`
* To demonstrate `sleep` and slightly also `editMessage` functions. >\
  `{{$x := sendMessageRetID nil "Hello"}} {{sleep 3}} {{editMessage nil $x "There"}} {{sleep 3}} {{sendMessage nil "We
  all know, that"}} {{sleep 3}} YAGPDB rules!`
* To demonstrate usage of `complexMessage` with `sendMessage`. `{{sendMessage nil (complexMessage "reply" .Message.ID
  "content" "Who rules?" "embed" (cembed "description" "YAGPDB of course!" "color" 0x89aa00) "file" "Here we print
  something nice - you all are doing awesome!" "filename" currentTime.Weekday)}}`
* To demonstrate usage of `complexMessageEdit` with `editMessage`.\
  `{{$mID := sendMessageRetID nil (complexMessage "content" "You know what is..." "silent" true "embed" (cembed "title"
  "FUN!?" "color" 0xaa8900))}} {{sleep 3}} {{editMessage nil $mID (complexMessageEdit "embed" (cembed "title" "YAGPDB!"
  "color" 0x89aa00) "content" "Yes, it's always working with...")}}{{sleep 3}}{{editMessage nil $mID (complexMessageEdit
  "embed" nil` "content" "Will delete this message in a sec, goodbye YAG!"`)}}{{deleteMessage nil $mID 3}}`

### Miscellaneous

{{% notice info %}}

`if`, `range`, `try-catch`, `while`, `with` actions are all covered [here](/reference/templates#actions).

{{% /notice %}}

<table data-header-hidden><thead><tr><th width="150">Function</th><th>Description</th></tr></thead><tbody><tr><td>**Function**</td><td>**Description**</td></tr><tr><td>`adjective`</td><td>Returns a random adjective.</td></tr><tr><td>`cembed` "list of embed values"</td><td>Function to generate embed inside custom command. [More in-depth here](/reference/custom-embeds). </td></tr><tr><td>`createTicket` author topic</td><td>Creates a new ticket with the author and topic provided. Covered in its own section [here](/reference/templates#tickets).</td></tr><tr><td>`cslice`, `sdict`</td><td>These functions are covered in their own section [here](/reference/templates#custom-types).</td></tr><tr><td>`dict` key1 value1 key2 value2 ...</td><td>Creates an unordered collection of key-value pairs, a dictionary so to say. The number of parameters to form key-value pairs must be even. Example [here](/reference/custom-command-examples#dictionary-example). Keys and values can be of any type. Key is not restricted to _string_ only as in case with `sdict`. `dict` also has helper methods .Del, .Get, .HasKey and .Set and they function the same way as `sdict` ones discussed [here](/reference/templates#templates.Sdict).</td></tr><tr><td>`exec` "command" "args" "args" "args" ...</td><td>Executes a YAGPDB command (e.g. roll, kick etc) in a custom command. Exec can be run max 5 times per CC. If real command returns an embed - `exec` will return raw data of type embed, so you can use embed fields for better formatting - e.g. `{{$resp := exec "whois"}} {{$resp.Title}} Joined at > {{(index $resp.Fields 4).Value}}` will return the title (username#discriminator) and "Joined at" field's value from `whois` command. <br>**NB!** This will not work for commands with paginated embed returns,  like `un/nn` commands!exec syntax is `exec "command" arguments` - this means you format it the same way as you would type the command regularly, just without the prefix, e.g. if you want to clear 2 messages and avoiding the pinned message > `{{exec "clear 2 -nopin"}}`, where `"command"` part is whole `"clear 2 -nopin"`. If you change that number inside CC somewhere then you have to use `arguments` part of exec formatting > `{{$x := 2}} {{exec "clear" $x "-nopin"}}` Here `"clear"` is the `"command"` and it is followed by `arguments`, one variable `$x` and one string `"-nopin"`. Last example is the same as `{{exec (joinStr " " "clear" $x "-nopin")}}`(also notice the space in `joinStr` separator). </td></tr><tr><td>`execAdmin` "command" "args" "args" "args" ...</td><td>Functions same way as `exec` but effectively runs the command as the bot user (YAGPDB). This has essentially the same effect as if a user with the same permissions and roles as YAGPDB ran the command: for example, if YAGPDB had ban members permission but the user which ran the command did not, `{{exec "ban" 12345}}` would error due to insufficient permissions but `{{execAdmin "ban" 12345}}` would succeed.</td></tr><tr><td>`execTemplate` "template" data</td><td>Executes the associated template, optionally with data. A more detailed treatment of this function can be found in the [Associated Templates](/reference/templates#associated-templates) section.</td></tr><tr><td>`getWarnings` user</td><td>Returns a _slice_ of warnings of type _[[]*moderation.WarningModel](https://github.com/botlabs-gg/yagpdb/blob/master/moderation/models.go#L121)_ given to user argument which can be its ID or user object.</td></tr><tr><td>`hasPrefix` string prefix</td><td>`hasPrefix` tests whether the given `string` begins with `prefix` and returns _bool_. Example > `{{hasPrefix "YAGPDB" "YAG"}}` returns `true`.</td></tr><tr><td>`hasSuffix` string suffix</td><td>hasSuffix tests whether the given `string` ends with `suffix` and returns _bool_.Example > `{{hasSuffix "YAGPDB" "YAG"}}` returns `false`.</td></tr><tr><td>`humanizeThousands` arg</td><td>This function places comma to separate groups of thousands of a number. `arg` can be _int_ or _string_, has to be a whole number, e.g. `{{humanizeThousands "1234567890"}}` will return `1,234,567,890`.</td></tr><tr><td>`in` list value</td><td>Returns _bool_ true/false whether case-sensitive value is in list/_slice_. `{{ in (cslice "YAGPDB" "is cool") "yagpdb" }}` returns `false`.</td></tr><tr><td>`index` arg ...keys</td><td>Returns the result by indexing its first argument with following arguments. Each indexed item must be a _map_, _slice_ or _array._ Indexed _string_ returns value in _uint8._ Example:  `{{index .Args 1}}` returns first argument after trigger which is always at position 0. More than one positional keys can be used, in pseudo-code: `index X 0 1` is equivalent to calling `index (index X 0) 1`</td></tr><tr><td>`inFold` list value</td><td>Same as `in`, but is case-insensitive. `{{inFold (cslice "YAGPDB" "is cool") "yagpdb"}}` returns `true`.</td></tr><tr><td>`kindOf` value (flag)</td><td>This function helps to determine what kind of data type we are dealing with. `flag` part is a _bool_ and if set as **true** (**false** is optional) returns the value where given `value` points to. Example: `{{kindOf cembed false}} {{kindOf cembed true}}` will return `ptr` and `struct`.</td></tr><tr><td>`len` arg</td><td>Returns the integer length of its argument. arg can be an _array_, _slice_, _map_, or _string._`{{ len (cslice 1 2 3) }}`returns `3`.</td></tr><tr><td>`noun`</td><td>Returns a random noun.</td></tr><tr><td>`parseArgs` required_args error_message `...carg`</td><td>Checks the arguments for a specific type. Has methods `.Get` and `.IsSet`. <br>`carg "type" "name"` is required by `parseArgs` and it defines the type of argument for `parseArgs`. An example in [Custom Command Examples.](/reference/custom-command-examples.md#parseargs-example)</td></tr><tr><td>`sendTemplate` channel templateName (data)</td><td>Function sends a formulated template to another channel and returns sent response's messageID.<br>Channel is like always either name, number or nil; and returns messageID. `data` is optional and is meant for additional data for the template.Example: `{{define "logsTemplate"}}This text will output on different channel, you can also use functions like {{currentTime}}. {{.TemplateArgs}} would be additional data sent out. {{end}}`Now we call that "logs" in the same custom command.`{{sendTemplate "logs" "logsTemplate" "YAG rules!"}}.`<br><br>Template definitions are discussed [here](https://pkg.go.dev/text/template#hdr-Nested_template_definitions).</td></tr><tr><td>`sendTemplateDM` templateName (data)</td><td>Works the same way as function above. Only channel's name is not in the arguments. YAGPDB **will only** DM the triggering user. This function will not work for leave messages.</td></tr><tr><td>`seq` start stop</td><td>Creates a new _slice_ of type _int_, beginning from start number, increasing in sequence and ending at stop (not included). `{{seq -4 2}}` returns a _slice_ `[ -4 -3 -2 -1 0 1 ]`. Sequence's max length is 10 000.</td></tr><tr><td>`shuffle` list</td><td>Returns a shuffled, randomized version of a list/_slice_.</td></tr><tr><td>`sleep` seconds</td><td>Pauses execution of template's action-structure inside custom command for max 60 seconds combined. Argument`seconds`is an integer (whole number). Example in [snippets](#message-sections-snippets).</td></tr><tr><td>`sort` slice (...args)</td><td>Sorts a slice of same type values with optional arguments. These arguments are presented in an `sdict` as keys: `"reverse"` true/false and `"key"` with dictionary/map's key name as value.Example > `{{sort (cslice (sdict "name" "bob") (sdict "name" "alice") (sdict "name" "yagpdb")) (sdict "key" "name" "reverse" true)}}` would return `[map[name:yagpdb] map[name:bob] map[name:alice]]`.Sorting mixed types is not supported. Previous sorting options `"subslices"` and `"emptyslices"` have been removed.<br>Sort function is limited to 1/3 CC calls regular/premium.</td></tr><tr><td>`verb`</td><td>Returns a random verb.</td></tr></tbody></table>

### Role functions

{{% notice info %}}

In all role functions where userID is required as argument to target a user, it can also be full user object.

{{% /notice %}}

|**Function**| **Description**|
|-| -|
|`addRoleID` roleID| Adds the role with the given ID to the user that triggered the command (use the `listroles` command for a list of roles).|
|`addRoleName` roleName| Adds the role with given name to the user that triggered the command (use the `listroles` command for a list of roles).|
|`getRole` role| Returns a [role object](https://discord.com/developers/docs/topics/permissions#role-object) of type _\*discordgo.Role._ `role`  can be either role's ID or role's name.|
|`giveRoleID` userID roleID| Gives a role by ID to the target.|
|`giveRoleName` userID "roleName"| Gives a role by name to the target.|
|`hasRoleID` roleID| Returns true if the triggering user has the role with the specified ID (use the listroles command for a list of roles).|
|`hasRoleName` "rolename"| Returns true if the triggering user has the role with the specified name (case-insensitive).|
|`removeRoleID` roleID (delay)| Removes the role with the given ID from the user that triggered the command (use the listroles command for a list of roles). `Delay` is optional argument in seconds.|
|`removeRoleName` roleName (delay)| Removes the role with given name from the user that triggered the command (use the listroles command for a list of roles). `Delay` is optional argument in seconds.|
|`roleAbove` role1 role2| `roleAbove` compares two role objects e.g. `getRole`return and gives`true/false` value is `role1` positioned higher than `role2` or not.|
|`setRoles` userID roles| Overwrites the roles of the given user using the slice of roles provided, which should be a slice of role IDs. IDs can be ints or strings. Example: `{{setRoles .User.ID cslice}}` would clear the roles of the triggering user.|
|`takeRoleID` userID roleID (delay)| Takes away a role by ID from the target. `Delay` is optional argument in seconds.|
|`takeRoleName` userID "roleName" (delay)| Takes away a role by name from the target. `Delay` is optional argument in seconds.|
|`targetHasRoleID` userID roleID| Returns true if the given/targeted user has the role with the specified ID (use the listroles command for a list of roles). Example in section's Snippets.|
|`targetHasRoleName` userID "roleName"| Returns true if the given/targeted user has the role with the specified name (case-insensitive).|

### String manipulation

{{% notice info %}}

All regexp functions are limited to 10 different pattern calls per CC.

{{% /notice %}}

|**Function**| **Description**|
|-| -|
|`joinStr` "separator" "str1" (arg1)(arg2) "str2" ...| Joins several strings into one, separated by the first argument`"separator"`, example:`{{joinStr "" "1" "2" "3"}}` returns `123`. Also if functions have _string, \[]string_ or easily convertible return, they can be used inside `joinStr` e.g. `{{joinStr ""` `"Let's calculate " (add (mult 13 3) 1 2) ", was returned at "` `(currentTime.Format "15:04") "."}}`|
|`lower` "string"| Converts the string to lowercase.|
|`print`, `printf`, `println`| These are GO template package's predefined functions and are aliases for [fmt.Sprint](https://golang.org/pkg/fmt/#Sprint), [fmt.Sprintf](https://golang.org/pkg/fmt/#Sprintf) and [fmt.Sprintln](https://golang.org/pkg/fmt/#Sprintln). Formatting is also discussed [here](https://golang.org/pkg/fmt/#hdr-Printing). `printf` cheat sheet [here](https://yourbasic.org/golang/fmt-printf-reference-cheat-sheet/).<br><br>`printf` is usable for example to determine the type of the value > `{{printf "%T" currentTime}}` outputs `currentTime` functions output value type of `time.Time`. In many cases, `printf` is a great alternative to `joinStr` for concatenate strings.|
|`reFind` "regex" "string"| Compares "string" to regex pattern and returns first match. `{{reFind "AG" "YAGPDB is cool!"}}`returns `AG` (regex pattern is case sensitive).|
|`reFindAll` "regex" "string" (count)| Adds all regex matches from the "string" to a _slice_. Example in section's [Snippets](functions.md#string-manipulation-sections-snippets). Optional `count` determines how many matches are made. **Example:** `{{reFindAll "a*" "abaabaccadaaae" 4}}` would return `[a aa a ].`|
|`reFindAllSubmatches` "regex" "string" (count)| Returns whole-pattern matches and also the sub-matches within those matches as _slices_ inside a _slice_. `{{reFindAllSubmatches "(?i)y([a-z]+)g" "youngish YAGPDB"}}` returns `[[young oun] [YAG A]]` (regex pattern here is case insensitive). Optional `count` works the same way as for `reFindAll`. So example above with `count` set to 1 would return `[[young oun]]`.|
|`reQuoteMeta` "string"| `reQuoteMeta` returns a string that escapes all regular expression metacharacters inside the argument text; the returned string is a regular expression matching the literal text. Example in [package documentation](https://pkg.go.dev/regexp#QuoteMeta).|
|`reReplace` "regex" "string1" "string2"| Replaces "string1" contents with "string2" at regex match point. `{{reReplace "I am" "I am cool!" "YAGPDB is"}}` returns  `YAGPDB is cool!` (regex pattern here is case sensitive). <br>Inside "string2" dollar-sign, $ with numeric name like $1 or ${1} are interpreted as referrals to the submatches in "regex" pattern, so for instance $1 represents the text of the first submatch. To insert a literal $ in the output, use $$.|
|`reSplit` "regex" "string" (count)| `reSplit` slices `string` into substrings separated by the `regex` expression and returns a _slice_ of the substrings between those expression matches. The optional `count` determines the number of substrings to return. If `count` is negative number the function returns all substrings, if 0 then none. If `count` is bigger than 0 it returns at most n substrings, the last substring being the unsplit remainder.**Example:** `{{ $x := reSplit "a" "yagpdb has a lot of fame" 5}}``{{$x}} {{index $x 3}}` would return `[y gpdb h s lot of f me]` and `lot of f.`|
|`sanitizeText` "string"| Detects accented and confusable characters in input and turns these characters to normal, ISO-Latin ones. `{{ sanitizeText "𝔭𝒶ỿ𝕡𝕒ℓ" }}`would return `paypal`.|
|`slice` arg integer (integer2)| Function's first argument must be of type _string_ or _slice_.Outputs the `arg` after cutting/slicing off integer (numeric) value of symbols (actually starting the string's index from integer through integer2) - e.g. `{{slice "Fox runs" 2}}`outputs `x runs`. When using also integer2 - e.g. `{{slice "Fox runs" 1 7}}`, it outputs `ox run`. For slicing whole arguments, let's say words, see example in section's [Snippets](#string-manipulation-sections-snippets). This `slice` function is not the same as basic dynamically-sized _slice_ data type discussed in this reference doc. Also it's custom, not having 3-indices as the default one from [text/template](https://golang.org/pkg/text/template/#hdr-Functions) package.|
|`split` "string" "sepr"| Splits given `"string"` to substrings separated by `"sepr"`arg and returns new _slice_ of the substrings between given separator e.g. `{{split "YAG, is cool!" ","}}` returns `[YAG  is cool!]` _slice_ where `YAG` is at `index` position 0 and `is cool!` at `index` position 1. Example also in section's [Snippets](functions.md#string-manipulation-sections-snippets).|
|`title` "string"| Returns the string with the first letter of each word capitalized.|
|`trimSpace` "string"| Returns the string with all leading and trailing white space removed.|
|`upper` "string"| Converts the string to uppercase.|
|`urlescape` "string"| Escapes the _string_ so it can be safely placed inside a URL path segment - e.g. "Hello, YAGPDB!" becomes "Hello%2C%20YAGPDB%21"<br>There's also predefined template package function `urlquery` which is covered [here](https://pkg.go.dev/text/template#hdr-Functions).|

{{% notice info %}}

Special information we can include in the string is _escape sequences_. Escape sequences are two (or more) characters,
the first of which is a backslash `\`, which gives the remaining characters special meaning - let's call them
metacharacters. The most common escape sequence you will encounter is `\n`, which means "newline".&#x20;

{{% /notice %}}

{{% notice info %}}

With regular expression patterns - when using quotes you have to "double-escape" metacharacters starting with backslash.
You can use backquotes/ticks to simplify this:`{{reFind "\\d+" (toString 42)}}` versus ``{{reFind `\d+` (toString
42)}}``

{{% /notice %}}

#### String manipulation section's snippets

* `{{$args:= (joinStr " " (slice .CmdArgs 1))}}` Saves all the arguments except the first one to a variable `$args`.&#x20;
* To demonstrate usage of `split` function. >\
  `{{$x := "Hello, World, YAGPDB, here!"}} {{range $k, $v := (split $x ", ")}}Word {{$k}}: __{{$v}}__ {{end}}`
* To demonstrate usage of `reFindAll`. > \
  `Before regex: {{$msg := "1 YAGPDB and over 100000 servers conquered."}} {{$re2 := reFindAll "[0-9]+" $msg}} {{$msg}}`  \
  `After regex matches: {{println "Only" (index $re2 0) "YAGPDB and already" (index $re2 1) "servers captured."}}`

### Time

|**Function**| **Description**|
|-| -|
|`currentTime`| Gets the current time, value is of type _time.Time_ which can be used in a custom embed. More info [here](/commands/custom-commands.md#currenttime-template).|
|`formatTime` Time ("layout arg")| Outputs given time in RFC822 formatting, first argument `Time` shows it needs to be of type _time.Time_, also with extra layout if second argument is given - e.g. `{{formatTime currentUserCreated "3:04PM"}}` would output `11:22AM` if that would have been when user was created. Layout argument is covered [here](https://pkg.go.dev/time#pkg-constants).|
|`humanizeDurationHours`| Returns given integer (whole number) or _time.Duration_ argument in nanoseconds in human readable format - as how long it would take to get towards given time - e.g. `{{humanizeDurationHours 9000000000000000000}}` returns `285 years 20 weeks 6 days and 16 hours`. More in [Snippets](functions.md#time-sections-snippets).|
|`humanizeDurationMinutes`| Same as `humanizeDurationHours`, this time duration is returned in minutes - e.g. `{{humanizeDurationMinutes 3500000000000}}` would return `58 minutes`.|
|`humanizeDurationSeconds`| Same as both humanize functions above, this time duration is returned in seconds - e.g. `{{humanizeDurationSeconds 3500000000000}}` would return `58 minutes and 20 seconds`.|
|`humanizeTimeSinceDays`| Returns time passed since given argument of type _time.Time_ in human readable format - e.g. `{{humanizeTimeSinceDays currentUserCreated}}.`|
|`loadLocation` "location"| Returns value of type _*time.Location_ which can be used further in other golang's [time](https://pkg.go.dev/time) functions, for example `{{currentTime.In (loadLocation "Asia/Kathmandu")}}` would return current time in Nepal. <br>`location` is of type _string_ and has to be in [ZONEINFO syntax](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).|
|`newDate` year month day hour minute second (timezone)| Returns type _time.Time_ object in UTC using given syntax (all required arguments need to be of type _int_), for example > `{{humanizeDurationHours ((newDate 2059 1 2 12 34 56).Sub currentTime)}}` will give you how much time till year 2059 January 2nd 12:34:56. More examples in [Snippets](#time-sections-snippets). `timezone` is an optional argument of type _string_ which uses golang's [LoadLocation](https://golang.org/pkg/time/#LoadLocation) function and [ZONEINFO syntax](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). For example: `{{newDate 2020 4 20 12 34 56 "Atlantic/Reykjavik"}}` would return that time in GMT+0.|
|`parseTime` value layout (location)| `parseTime` uses golang's [parseInLocation](https://pkg.go.dev/time#ParseInLocation) time function and returns value of type _time.Time_. Argument `value` is the time representation that needs to be parsed and has to be a string which matches and is formatted using `layout` argument. `layout` must be either a slice of strings or a single string and max number of layouts is 50. [Layout reference](https://pkg.go.dev/time#pkg-constants).<br> `location` is optional, defaulting to UTC; of type _string_ and has to be in [ZONEINFO syntax](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).<br><br>For example: `{{parseTime "July 18, 2016 at 12:38am" "January 2, 2006 at 3:04pm" "Asia/Kathmandu"}}` would return that time in +0545.|
|`snowflakeToTime` snowflake| Converts given [snowflake](https://discord.com/developers/docs/reference#snowflakes) to type _time.Time_ e.g. using bot's ID `{{snowflakeToTime .BotUser.ID}}` returns `2016-07-17 15:17:19 +0000 UTC`for YAGPDB.|
|`timestampToTime` arg| Converts UNIX timestamp to _time.Time_. Example: \{{timestampToTime 1420070400\}} would return same time as `.DiscordEpoch`.|
|`weekNumber` time| Returns the week number as _int_ of given argument `time` of type _time.Time_. `{{weekNumber currentTime}}` would return the week number of current time.|

{{% notice info %}}

Discord Timestamp Styles referenced
[here](https://discord.com/developers/docs/reference#message-formatting-timestamp-styles) can be done using `print`
function e.g.

`{{print "<t:" currentTime.Unix ":F>"}}` for "Long Date/Time" formatting.

{{% /notice %}}

#### Time section's snippets

* To demonstrate `humanizeDurationHours` and also how to parse a timestamp, output will be like `whois` command shows
  user's _join server age_.\
  `{{humanizeDurationHours (currentTime.Sub .Member.JoinedAt.Parse)}}`
* To demonstrate `newDate` to get Epoch times.\
  `{{$unixEpoch := newDate 1970 1 1 0 0 0}} in seconds > {{$unixEpoch.Unix}}`\
  `{{$discordEpoch := newDate 2015 1 1 0 0 0}} in seconds > {{$discordEpoch.Unix}}`

### Type conversion

<table data-header-hidden><thead><tr><th width="150">Function</th><th>Description</th></tr></thead><tbody><tr><td>**Function**</td><td>**Description**</td></tr><tr><td>`json` value (flag)</td><td>Traverses given `value` through MarshalJSON ([more here](https://golang.org/pkg/encoding/json/#Marshal)) and returns it as type _string_. For example `{{json .TimeHour}}` outputs type _string_; before this `.TimeHour` was of type _time.Duration_. Basically it's good to use if multistep type conversion is needed `(toString (toInt value) )` and certain parts of `cembed` need this for example. `flag` part is a _bool_ and if set as **true** (**false** is optional) returns the value indented through MarshalIndent.</td></tr><tr><td>`jsonToSdict` value</td><td>Returns `templates.SDict` from given JSON argument, e.g. `{{jsonToSdict `{"yagpdb":"is cool"}` }}` would print `map[yagpdb:is cool]`.</td></tr><tr><td>`structToSdict` struct</td><td>Function converts exported field-value pairs of a _struct_ to an _sdict_. For example it is useful for editing embeds, rather than having to reconstruct the embed field by field manually. Example: `{{$x := cembed "title" "Something rules!" "color" 0x89aa00}} {{$x.Title}} {{$x = structToSdict $x}} {{- $x.Set "Title" "No, YAGPDB rules!!!" -}} {{$x.Title}} {{$x}}` will return No, YAGPDB rules!!! and whole sdict-mapped _cembed_.</td></tr><tr><td>`toByte` "arg"</td><td>Function converts input to a slice of bytes - meaning _[]uint8_. `{{toByte "YAG€"}}` would output `[89 65 71 226 130 172]`. `toString` is capable of converting that slice back to _string_.</td></tr><tr><td>`toDuration`</td><td>Converts the argument, number or string to type _time.Duration_ - more duration related methods [here](https://pkg.go.dev/time#Duration). Number represents nanoseconds. String can be with time modifier (second, minute, hour, day etc) `s, m, h, d, w, mo, y`,without a modifier string will be converted to minutes. Usage:`(toDuration x)`. Example in section's [Snippets](#type-conversion-sections-snippets). </td></tr><tr><td>`toFloat`</td><td>Converts argument (_int_ or _string_ type of a number) to type _float64_.  Usage: `(toFloat x)`. Function will return 0, if type can't be converted to _float64_.</td></tr><tr><td>`toInt`</td><td>Converts something into an integer of type _int_. Usage: `(toInt x)`. Function will return 0, if type can't be converted to _int._</td></tr><tr><td>`toInt64`</td><td>Converts something into an _int64_. Usage: `(toInt64 x)`.  Function will return 0, if type can't be converted to _int64._</td></tr><tr><td>`toRune` "arg"</td><td>Function converts input to a slice of runes - meaning _[]int32_. `{{toRune "YAG€"}}`would output `[89 65 71 8364]`. These two functions - the one above, are good for further analysis of Unicode strings. `toString` is capable of converting that slice back to _string_.</td></tr><tr><td>`toString`</td><td>Has alias `str`. Converts some other types into a _string_. Usage: `(toString x)`.</td></tr></tbody></table>

#### Type conversion section's snippets

* To demonstrate `toDuration`, outputs 12 hours from current time in UTC.\
  `{{(currentTime.Add (toDuration (mult 12 .TimeHour))).Format "15:04"}}`is the same as`{{(currentTime.Add (toDuration "12h")).Format "15:04"}}` or`{{(currentTime.Add (toDuration 43200000000000)).Format "15:04"}}`

{{% notice style="tip" %}}

**Tip:** You can convert a Unicode code point back to its string equivalent using `printf "%c"`.  For example, `printf
"%c" 99` would result in the string `c` as `99` is the Unicode code point for `c`.`printf` is briefly covered later on
in the next section, further documentation can be found [here.](https://golang.org/pkg/fmt/) Cheat sheet
[here](https://yourbasic.org/golang/fmt-printf-reference-cheat-sheet/).

{{% /notice %}}

### User

|**Function**| **Description**|
|-| -|
|`currentUserAgeHuman`| The account age of the current user in more human readable format.|
|`currentUserAgeMinutes`| The account age of the current user in minutes.|
|`currentUserCreated`| Returns value of type _time.Time_ and shows when the current user was created.|
|\~`pastNicknames` userID offset| Same as `pastUsernames`.|
|\~`pastUsernames` userID offset| Returns a _slice_ of type _[ ]*logs.CCNameChange_ having fields .Name and .Time of previous 15 usernames and skips `offset` number in that list.`{{range pastUsernames .User.ID 0}}` <br>`{{.Name}} - {{.Time.Format "Jan _2 2006"}}` <br>`{{end}}` |
|`userArg` mention/userID| Function that can be used to retrieve .User object from a mention or userID.`{{(userArg .User.ID).Mention}}` mentions triggering user. Explained more in [this section's snippets](#user-sections-snippets). Previous limit of 5 to this function is no longer there.|

#### User section's snippets

`{{(userArg .Guild.OwnerID).String}}` this template's action-structure returns Guild/Server owner's username and
discriminator as of type _string_. First, `userArg` function is given `.Guild.OwnerID` as argument (what it does,
explained in [templates section](/reference/templates#guild-server)). The parentheses surrounding them make `userArg` function return
`.User` as [.User object](/reference/templates#user) which is handled further by `.String` method (ref.`.User.String`), giving a result
like > `YAGPDB#8760`.