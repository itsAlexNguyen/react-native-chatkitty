# react-native-chatkitty

Build text chat and video/audio calls into your React Native application with ChatKitty.

## Installing the ChatKitty React Native SDK

To use the ChatKitty React Native Calls SDK, you'll need to add the [ChatKitty React Native SDK](https://www.npmjs.com/package/react-native-chatkitty) NPM package to your React Native project:

```
yarn add react-native-chatkitty
```

## Initializing the ChatKitty React Native SDK with your API key

With your API key from the[ ChatKitty dashboard](https://dashboard.chatkitty.com/authorization/register), you can initialize a new instance of the ChatKitty client:

```javascript
import ChatKitty from 'react-native-chatkitty';

const kitty = ChatKitty.getInstance('YOUR CHATKITTY API KEY HERE');
```

## Starting a guest user session

You must create a [**user session**](https://docs.chatkitty.com/concepts/user-sessions.md) before a user can begin chatting with other users using ChatKitty. A user session represents a secure bi-directional connection with ChatKitty servers allowing users to send and receive messages in real-time.

Before you learn about authenticating users with ChatKitty, you can create a guest user session. You can start a user session by passing a unique **username** to your `ChatKitty` client `startSession()` method. A username is a string that uniquely identifies a user within your application.

```typescript
import {
  failed,
  StartedSessionResult,
  succeeded,
} from 'react-native-chatkitty';

const result = await kitty.startSession({
  username: 'jane@chatkitty.com',
});

if (succeeded<StartedSessionResult>(result)) {
  const session = result.session; // Handle session
}

if (failed(result)) {
  const error = result.error; // Handle error
}
```

## Initializing the React Native Calls SDK context

Before you can begin using the ChatKitty React Native Calls SDK, you'll need to initialize the `ChatKitty.Calls` context object on your `ChatKitty` instance.

```javascript
await kitty.Calls.initialize({
  media: { audio: true, video: true },
});
```

After calling `initialize(...)`, the `ChatKitty.Calls` object creates a WebRTC MediaSteam for your device's camera and is ready to start and answer calls.

## Retrieving the local Media Stream

`kitty.Calls.localStream` exposes the React Native WebRTC MediaStream capturing video/audio from your device.

```tsx
import React from 'react';
import { MediaStream, RTCView } from 'react-native-webrtc';

const Snippet = () => {
  const localStream: MediaStream | null = kitty.Calls.localStream;

  return (
    localStream && (
      <RTCView objectFit="cover" streamURL={localStream.toURL()} zOrder={1} />
    )
  );
};
```

## Displaying Media Streams

To display a `MediaStream` (local or remote), use the React Native WebRTC `RTCView` component.

```jsx
 <RTCView
  style={styles.myStream}
  objectFit="cover"
  streamURL={stream.toURL()}
  zOrder={1}
/>;
```

## Retrieving online users

You can fetch a list of other users that are online.

```typescript
import {
  GetUsersSucceededResult,
  succeeded,
  User,
} from 'react-native-chatkitty';

const result = await kitty.getUsers({ filter: { online: true } });

if (succeeded<GetUsersSucceededResult>(result)) {
  const users: User[] = result.paginator.items;
}
```

## Observing user online/presence changes

You can listen to changes to user presence changes across your application - when users come online or go offline.&#x20;

```typescript
import { User, UserPresence } from 'react-native-chatkitty';

kitty.onUserPresenceChanged(async (user: User) => {
  const presence: UserPresence = user.presence;

  // Update online users list
});
```

## Observing call events

You can observe events related to calls by registering event listeners on the `ChatKitty.Calls` object.

### On call invite

Called when another user invites the current user to a call.

```typescript
import { Call } from 'react-native-chatkitty';

kitty.Calls.onCallInvite((call: Call) => {
  // Inform the current user about the call then accept or reject the call
});
```

### On call active

Called when the current user starts a call or accepts an incoming call and their is device ready.&#x20;

```typescript
import { Call } from 'react-native-chatkitty';

kitty.Calls.onCallActive((call: Call) => {
  // Update in-call state and navigate to in-call UI
});
```

### On participant accepted call

Called when another user accepts the call the current user is currently active in.

```typescript
import { User } from 'react-native-chatkitty';

kitty.Calls.onParticipantAcceptedCall((participant: User) => {
  // Update in-call state and UI
});
```

### On participant rejected call

Called when another user rejects the call the current user is currently active in.

```typescript
import { User } from 'react-native-chatkitty';

kitty.Calls.onParticipantRejectedCall((participant: User) => {
  // Update in-call state and UI
});
```

### On participant active

Called when another user's device is ready to send their video/audio stream and interact with the call. &#x20;

```typescript
import { User } from 'react-native-chatkitty';
import { MediaStream } from 'react-native-webrtc';

kitty.Calls.onParticipantActive((participant: User, stream: MediaStream) => {
  // Update in-call state and UI
});
```

### On participant left call

Called when another user leaves the call.

```typescript
import { User } from 'react-native-chatkitty';
import { MediaStream } from 'react-native-webrtc';

kitty.Calls.onParticipantLeftCall((participant: User) => {
  // Update in-call state and UI
});
```

### On call ended

Called when this call has ended.

```typescript
import { Call } from 'react-native-chatkitty';

kitty.Calls.onCallEnded((call: Call) => {
  // Update state and exit in-call UI
});
```

## Starting a call

Start a direct call with another user.

```javascript
await kitty.Calls.startCall({ members: [{ username: 'john@chatkitty.com' }] });
```

## Accepting a call

Accept a call invite.

```javascript
await kitty.Calls.acceptCall({ call });
```

## Rejecting a call

Reject a call invite.

```javascript
await kitty.Calls.rejectCall({ call });
```

## Leaving an active call

Leave the currently active call. Ends a one-to-one direct call.

```javascript
kitty.Calls.leaveCall();
```

## Switching camera

Switches the current user's camera if their device has multiple cameras (front and back).

```javascript
kitty.Calls.switchCamera();
```

## Toggling mute

Mutes/unmutes the current user's audio stream

```javascript
kitty.Calls.toggleMute();
```

## Retrieving active call mute state

Retrieve the active call's mute state using `ChatKitty.Calls.isMuted`.

```typescript
const muted: boolean = kitty.Calls.isMuted;
```

## Retrieving calls

Retrieve past calls.

```typescript
import {
  Call,
  GetCallsSucceededResult,
  succeeded,
} from 'react-native-chatkitty';

const result = await kitty.Calls.getCalls({
  channel,
  filter: { active: false },
});

if (succeeded<GetCallsSucceededResult>(result)) {
  const calls: Call[] = result.paginator.items;
}
```

## Retrieving a call

### Retrieving active call

Retrieve an active call if the currrent user has started or accepted the call using `ChatKitty.Calls.activeCall`

```typescript
import { Call } from 'react-native-chatkitty';

const call: Call | null = kitty.Calls.activeCall;
```

### Retrieving a call by ID

Retrieve a call with its ID.

```typescript
import {
  Call,
  GetCallSucceededResult,
  succeeded,
} from 'react-native-chatkitty';

const result = await kitty.Calls.getCall(id);

if (succeeded<GetCallSucceededResult>(result)) {
  const call: Call = result.call;
}
```

## Closing the React Native Calls SDK context

Close the ChatKitty React Native Calls SDK and clean up associated system resources

```javascript
kitty.Calls.close();
```

## Ending the user session

[End the user session](https://docs.chatkitty.com/concepts/user-sessions.md#ending-a-user-session) to log the user out, close their device's concurrent connection to ChatKitty and free up resources used by the ChatKitty SDK.

```javascript
kitty.endSession();
```

## Contributing

See the [contributing guide](CONTRIBUTING.md) to learn how to contribute to the repository and the development workflow.

## License

MIT
