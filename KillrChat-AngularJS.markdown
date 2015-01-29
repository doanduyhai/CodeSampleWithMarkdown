```javascript

killrChat.controller('ListAllRoomsCtrl', function($scope, ListAllRoomsService){

    $scope.allRooms = [];

    $scope.joinRoom = function(roomToJoin) {
        ListAllRoomsService.joinRoom($scope, roomToJoin);
    };

    $scope.quitRoom = function(roomToLeave) {
        ListAllRoomsService.quitRoom($scope, roomToLeave);
    };

    $scope.$evalAsync(ListAllRoomsService.loadInitialRooms($scope));
});

```


```javascript
killrChat.service('RememberMeService',function($rootScope, $location, $cookieStore, RememberMe) {...});

killrChat.service('GeneralNotificationService', function($rootScope) {...});

killrChat.service('SecurityService', function($rootScope, $location, $cookieStore, User) {...});

killrChat.service('UserRoomsService', function(Room, ParticipantService, GeneralNotificationService) {...});

killrChat.service('ParticipantService', function() {...});

killrChat.service('NavigationService', function(Room, ParticipantService, UserRoomsService, GeneralNotificationService) {...});

killrChat.service('WebSocketService', function(ParticipantService, UserRoomsService, GeneralNotificationService) {...});

killrChat.service('ChatService', function(Message, WebSocketService, GeneralNotificationService) {...});

killrChat.service('RoomService', function(ParticipantService) {...});

killrChat.service('ListAllRoomsService', function(Room, RoomService, ParticipantService, GeneralNotificationService) {...});

killrChat.service('RoomCreationService', function(Room) {...});
```

```javascript
killrChat.factory('User', function($resource) {
    return $resource('users', [],{
        'create': {url: 'users', method: 'POST', isArray:false, headers:{'Content-Type': 'application/json'}},
        'login': {url: 'authenticate', method: 'POST', isArray:false, headers: {'Content-Type': 'application/x-www-form-urlencoded'}},
        'load': {url : 'users/:login', method:'GET', isArray:false, headers:{'Accept': 'application/json'}},
        'logout': {url: 'logout', method: 'GET', isArray:false, headers:{'Accept': 'application/json'}}
    });
});
```

```javascript
killrChat.factory('Room', function($resource) {
    return $resource('rooms', [],{
        'create': {url: 'rooms/:roomName', method: 'POST', isArray:false,...},
        'delete': {url: 'rooms/:roomName', method: 'PATCH', isArray:false,...},
        "addParticipant": {url: 'rooms/participant/:roomName', method: 'PUT', isArray:false,...},
        'removeParticipant': {url: 'rooms/participant/:roomName', method: 'PATCH', isArray:false,...},
        'load': {url: 'rooms/:roomName', method: 'GET', isArray:false,...},
        'list': {url: 'rooms', method: 'GET', isArray:true,...}
    });
});
```

```javascript
 Room.load({roomName:roomToEnter})
            .$promise
            .then(function(...){...})
            .catch(function(...){...});
```

```javascript
 new User().$login({
            j_username: $scope.username,
            j_password: $scope.password,
            _spring_security_remember_me: $scope.rememberMe
        })
        .then(function() {...})
        .catch(function(...){...});
```

```javascript
killrChat.directive('passwordMatch', function() {
    return {
        require: 'ngModel',
        ...,
        link: function (scope, el, attrs, ngCtrl) {
            scope.$watch(function(){
                // check for equality in the watcher and return validity flag
                var modelValue = ngCtrl.$modelValue;
                return (ngCtrl.$pristine && angular.isUndefined(modelValue)) 
                       || angular.equals(modelValue, scope.passwordMatch);
            },function(validBoolean){
                // set validation with validity in the listener
                ngCtrl.$setValidity('passwordMatch', validBoolean);
            });
        }
    }
}  
```

```html
 <!--Chat Main Section-->
  <chat-zone state="state" user="user" home="home()" in-room="inRoom()" get-light-model="getLightModel()"></chat-zone>
```

```javascript
killrChat.directive('chatZone', function(...) {
    return {
        ...,
        templateUrl: 'views/templates/chatWindow.html',
        scope: {
            state: '=',
            user: '=',
            home: '&',
            getLightModel: '&'
        },
        controller: 'ChatCtrl',
        link: function (scope, root) {
            ...
            //Change in the list of chat messages should be intercepted
            scope.$watchCollection(
                function() {  // watch on chat message
                    return scope.messages;
                },
                function(newMessages,oldMessages) { // on change of chat messages
                    if(scrollMode === 'display') {
                        if(newMessages.length > oldMessages.length){
                            element.scrollTop = element.scrollHeight;
                        }
                    } else if(scrollMode === 'loading') {
                        element.scrollTop = 10;
                    }
                }
            );

            wrappedElement.bind('scroll', function() {
                if( <!-- user scrolls down --> ) {
                    scrollMode = 'display';
                } else if( <!-- user scroll up && more data to load --> ) {
                    scope.$apply(function(){
                        usSpinnerService.spin('loading-spinner');
                        scrollMode = 'loading';
                        ChatService.loadPreviousMessages(scope)
                        .then(function(messages){
                            // if no more message found, stop loading messages on next calls
                            if(messages.length == 0) {
                                loadMoreData = false;
                            }
                            usSpinnerService.stop('loading-spinner');
                        });
                    });

                } else {
                    scrollMode = 'fixed';
                }
            });
            ...
        }
    }
});
```

```javascript
    this.initSockets = function($scope) {
        self.closeSocket($scope);
        var roomName = $scope.state.currentRoom.roomName;
        $scope.socket.client = new SockJS('/killrchat/chat');
        var stomp = Stomp.over($scope.socket.client);
        stomp.debug = function(str) {};
        stomp.connect({}, function() {
            stomp.subscribe('/topic/messages/'+roomName,
                function(message){ self.notifyNewMessage($scope,message) });
            stomp.subscribe('/topic/participants/'+roomName,
                function(message) { self.notifyParticipant($scope, message) });
            stomp.subscribe('/topic/action/'+roomName,
                function(message) { self.notifyRoomAction($scope, message) });
        });
        $scope.socket.stomp = stomp;
    };
```

```java
@Inject
private SimpMessagingTemplate template;

@RequestMapping(value = "/{roomName}", method = POST, consumes = APPLICATION_JSON_VALUE)
@ResponseStatus(HttpStatus.NO_CONTENT)
public void postNewMessage(@PathVariable String roomName, @NotNull @RequestBody @Valid MessagePosting messagePosting) throws JsonProcessingException {
    ...
    template.convertAndSend("/topic/messages/"+roomName, messageModel);
}

...
@RequestMapping(value = "/participant/{roomName}", method = PUT, consumes = APPLICATION_JSON_VALUE)
@ResponseStatus(HttpStatus.NO_CONTENT)
public void addUserToChatRoom(@PathVariable @NotEmpty String roomName, @NotNull @RequestBody @Valid LightUserModel participant) {
    chatRoomService.addUserToRoom(roomName, participant);
    final MessageModel joiningMessage = messageService.createJoiningMessage(roomName, participant);
    template.convertAndSend("/topic/participants/"+ roomName, participant, ImmutableMap.<String,Object>of("status", Status.JOIN));
    template.convertAndSend("/topic/messages/"+roomName, joiningMessage);
}

@RequestMapping(value = "/participant/{roomName}", method = PATCH, consumes = APPLICATION_JSON_VALUE)
@ResponseStatus(HttpStatus.NO_CONTENT)
public void removeUserFromChatRoom(@PathVariable @NotEmpty String roomName, @NotNull @RequestBody @Valid LightUserModel participant) {
    chatRoomService.removeUserFromRoom(roomName, participant);
    final MessageModel leavingMessage = messageService.createLeavingMessage(roomName, participant);
    template.convertAndSend("/topic/participants/"+ roomName, participant, ImmutableMap.<String,Object>of("status", Status.LEAVE));
    template.convertAndSend("/topic/messages/"+roomName, leavingMessage);
}
```

```javascript
killrChat.service('ParticipantService', function(){
    var self = this;
    this.sortParticipant = function(participantA,participantB){
        return participantA.firstname.localeCompare(participantB.firstname);
    };

    this.addParticipantToCurrentRoom = function(currentRoom, participantToAdd) {
        currentRoom.participants.push(participantToAdd);
        currentRoom.participants.sort(self.sortParticipant);
    };

    this.removeParticipantFromCurrentRoom = function(currentRoom, participantToRemove) {
        var indexToRemove = currentRoom.participants.map(function(p){return p.login}).indexOf(participantToRemove.login);
        currentRoom.participants.splice(indexToRemove, 1);
    };
});
```

```html
<input type="password" class="form-control" name="password" 
    placeholder="Password" 
    ng-model="user.password" required>

<input type="password" class="form-control" name="confirm_password" 
    placeholder="Password Confirm" 
    password-match="user.password" 
    ng-model="user.passwordConfirm" required>
```

```javascript

```

```javascript

```

```javascript

```
