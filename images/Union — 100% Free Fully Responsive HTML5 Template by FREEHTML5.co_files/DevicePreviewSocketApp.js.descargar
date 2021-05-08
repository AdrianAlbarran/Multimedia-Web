/* Copyright 2015 Adobe Systems Incorporated.  All rights reserved. */
/*jslint nomen: true */
/*global io, console, $, URI*/
function SocketApp() {
    //member variables
    'use strict';
    this._socket = null;
    this._initialized = false;
    this._urlPreviewed = '';
    this._isServerPreviewed = false;
    this._setURLSourceTimer = null;
    /*********************************************************************************************
    //get/set functions
    *********************************************************************************************/
    /*
        This function returns the socket object connected with the preview service.
    */
    this.getSocket = function () {
        return this._socket;
    };
    /*
        Initialization of socket and all the handlers to
        communicate with the preview service.
    */
    this.initWebsocket = function () {
        var userId = window.preview.main.getUserId(),
            sessionId = window.preview.main.getSessionId(),
            hostURL,
            devicePixelRatio;
        
		if (this._socket) {
            this._socket.io.reconnect();
        } else {
			hostURL = window.preview.config.DW_SERVER_PROTOCOL + '://' + window.preview.config.DW_SERVER_HOST
                + ':' + window.preview.config.DW_SERVER_PORT + '/';
            devicePixelRatio = window.devicePixelRatio;
            
			if (userId && userId !== '') {
				this._socket = io.connect(hostURL, { 'sync disconnect on unload': true, query: "userid=" + userId + "&dpxr=" + devicePixelRatio + "&sessionid=" + sessionId});
			} else {
				this._socket = io.connect(hostURL);
			}
        }
    };
    
    /*
        Disconnect the socket.
    */
    this.teardown = function () {
        this.removeSocketHandlers();
        this._socket.io.disconnect();
        this._initialized = false;
        this._isServerPreviewed = false;
    };
    
    this.removeSocketHandlers = function () {
        var socket = this.getSocket();
        socket.removeAllListeners('connect');
        socket.removeAllListeners('check:authorization:cb');
        socket.removeAllListeners('scroll:dw');
        socket.removeAllListeners('join:room:cb');
        socket.removeAllListeners('url:change:cb');
        socket.removeAllListeners('mouse:in');
        socket.removeAllListeners('mouse:out');
        socket.removeAllListeners('partial:refresh:delete:rule');
        socket.removeAllListeners('partial:refresh:insert:rule');
        socket.removeAllListeners('query:device:viewport');
        socket.removeAllListeners('inspect:dw');
        socket.removeAllListeners('inspect:off:dw');
    };
 
    this.updateIframeUrl = function (url, isServer) {
        var self = this;
        if (isServer) {
            this._urlPreviewed = url;
            this._isServerPreviewed = true;
        } else {
            this._urlPreviewed = window.preview.utils.getDwSocketAppUrl() + url;
            if (this._urlPreviewed && this._urlPreviewed.indexOf('DP_CLIENT_SOCKET_ID') !== -1) {
                this._urlPreviewed = this._urlPreviewed.replace('DP_CLIENT_SOCKET_ID', URI.encode(this.getSocket().id));
            }
            this._isServerPreviewed = false;
        }

        $('#previewApp')[0].src = '';
        if (this._setURLSourceTimer) {
            window.clearTimeout(this._setURLSourceTimer);
        }
        
        this._setURLSourceTimer = window.setTimeout(function () {
            var v = Math.floor((Math.random() * 1000) + 1);
            $('#previewApp')[0].src = self._urlPreviewed + "?v=" + v;
            $('#previewApp')[0].contentWindow.location.href = self._urlPreviewed;
        }, 50);
    };
    
    this.initSocketCallbacks = function () {
        var socket = this.getSocket(),
            userId = window.preview.main.getUserId(),
            self = this;

        socket.on('connect', function () {
            socket.emit('check:authorization', userId);
        });
        
		socket.on('check:authorization:cb', function (isauhtorized) {
			if (isauhtorized) {
				socket.emit("join:room", userId);
			}
		});

        socket.on('url:change:cb', function (data) {
            self.updateIframeUrl(data.url, data.isServer);
        });
        
        socket.on('disconnect:yourself', function (url) {
            self.updateIframeUrl(url);
            self.teardown();
        });
        
        socket.on('scroll:dw', function (scrollParams) {
            if (!self._isServerPreviewed) {
                if (window.preview.main.getIsIOSDevice()) {
                    //we do not support sync scroll in IOS devices
                    return;
                }

                var iframe = document.getElementById('previewApp'),
                    scrollData = {
                        'event': 'scroll',
                        'xperc': scrollParams.xperc,
                        'yperc': scrollParams.yperc
                    };
                iframe.contentWindow.postMessage(scrollData, iframe.src);
            }
        });
        
        socket.on('join:room:cb', function (data) {
            if (data.success === "-1") {
                console.log("Disconnected. Couldn't join the Group.");
            } else {
                var urlPreviewed = data.urlPreviewed;
                if (urlPreviewed) {
                    self.updateIframeUrl(urlPreviewed, data.isServer);
                }
                
                window.preview.overlayStrings.OVERLAY_STRING_DEVICE_PIXEL_RATIO = data.overlayString_DevicePixelRatio;
                window.preview.overlayStrings.OVERLAY_STRING_OS = data.overlayString_OS;
                window.preview.overlayStrings.OVERLAY_STRING_VIEWPORT_SIZE = data.overlayString_ViewportSize;
                window.preview.overlayStrings.OVERLAY_STRING_USER_AGENT = data.overlayString_UserAgent;
                document.title = data.document_title;
            }
            self.logConnectionSuccessEvent();
        });
        
        socket.on('mouse:in', function (data) {
            var deviceModel = data.device && data.device.model ? data.device.model : "",
                deviceVendor = data.device && data.device.vendor ? data.device.vendor : "",
                osName = data.os && data.os.name ? data.os.name : "",
                osVersion = data.os && data.os.version ? data.os.version : "",
                browserName = data.browser && data.browser.name ? data.browser.name : "",
                browserVersion = data.browser && data.browser.major ? '( ' + data.browser.major + ' )' : "",
                devicePixelRatio = data.devicePixelRatio || '',
                containerWidth,
                containerHeight,
                marginTop,
                marginLeft,
                viewportWidth,
                viewportHeight,
                viewportSize,
                eventParams;
           
            if (deviceModel) {
                $('.deviceInfo').html(deviceVendor + ' ' + deviceModel);
            }
            if (osName) {
                $('.osInfo').html(window.preview.overlayStrings.OVERLAY_STRING_OS + ' : ' + osName + ' ' + osVersion);
            }
            if (browserName) {
                $('.browserInfo').html(window.preview.overlayStrings.OVERLAY_STRING_USER_AGENT + ' : ' + browserName + ' ' + browserVersion);
            }
           
            viewportWidth = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
            viewportHeight = Math.max(document.documentElement.clientHeight, window.innerHeight || 0);
            viewportSize = viewportWidth + ' x ' + viewportHeight;
            
            $('.viewportSize').html(window.preview.overlayStrings.OVERLAY_STRING_VIEWPORT_SIZE + ' : ' + viewportSize);
            
            if (devicePixelRatio) {
                devicePixelRatio = parseFloat(Number((devicePixelRatio)).toFixed(2));
                $('.devicePixelRatio').html(window.preview.overlayStrings.OVERLAY_STRING_DEVICE_PIXEL_RATIO + ' : ' + devicePixelRatio);
            }
            
            $('.overlay').show();
           
            eventParams = {
                eventName: "metadata-overlay-shown",
                params: {}
            };
        
            self.logPreviewEvent(eventParams);
        });
        
        socket.on('mouse:out', function (data) {
            $('.overlay').hide();
        });
        
        socket.on('partial:refresh:delete:rule', function (rule) {
            if (!self._isServerPreviewed) {
                if (rule) {
                    rule.encodedPath = window.preview.utils.convertPathArrayToString(rule.pathArray);
                }
                var iframe = document.getElementById('previewApp'),
                    partialRefreshEvent = {
                        'event': 'partialRefreshDeleteRule',
                        rule: rule
                    };
                iframe.contentWindow.postMessage(partialRefreshEvent, self._urlPreviewed);
            }
        });

        socket.on('partial:refresh:insert:rule', function (rule) {
            if (!self._isServerPreviewed) {
                if (rule) {
                    rule.encodedPath = window.preview.utils.convertPathArrayToString(rule.pathArray);
                }
                var iframe = document.getElementById('previewApp'),
                    partialRefreshEvent = {
                        'event': 'partialRefreshInsertRule',
                        rule: rule
                    };
                iframe.contentWindow.postMessage(partialRefreshEvent, self._urlPreviewed);
            }
        });
        
        socket.on('query:device:viewport', function () {
            var viewportWidth = Math.max(document.documentElement.clientWidth, window.innerWidth || 0),
                viewportHeight = Math.max(document.documentElement.clientHeight, window.innerHeight || 0),
                viewportSize = {
                    width: viewportWidth,
                    height: viewportHeight
                };
            socket.emit('query:device:viewport:cb', viewportSize);
        });
        
        socket.on('partial:refresh:HTML', function (argObj) {
            if (!self._isServerPreviewed) {
                var iframe = document.getElementById('previewApp'),
                    partialRefreshEvent = {
                        'event': 'partialRefreshHTML',
                        args: argObj
                    };
                iframe.contentWindow.postMessage(partialRefreshEvent, self._urlPreviewed);
            }
        });

        socket.on('scroll:element:view', function (data) {
            if (!self._isServerPreviewed) {
                var iframe = document.getElementById('previewApp'),
                    inspectEvent = {
                        'event': 'scrollElement',
                        'liveEditId': data.liveId,
                        'selectorString': data.selectorString
                    };
                iframe.contentWindow.postMessage(inspectEvent, self._urlPreviewed);
            }
        });
        
        socket.on('inspect:dw', function (data) {
            if (!self._isServerPreviewed) {
                var iframe = document.getElementById('previewApp'),
                    inspectEvent = {
                        'event': 'inspect',
                        'liveEditId': data.liveId,
                        'selectorString': data.selectorString
                    };
                iframe.contentWindow.postMessage(inspectEvent, self._urlPreviewed);
            }
        });

        socket.on('inspect:off:dw', function () {
            if (!self._isServerPreviewed) {
                var iframe = document.getElementById('previewApp'),
                    inspectEvent = {
                        'event': 'inspectOff'
                    };
                iframe.contentWindow.postMessage(inspectEvent, self._urlPreviewed);
            }
        });
	};
    
    this.logConnectionSuccessEvent = function () {
        var accessMethod = window.preview.utils.getAccessMethod(),
            eventName,
            eventParams;
        
        if (accessMethod === 'c') {
            eventName = 'connection-success-short-link';
        } else if (accessMethod === 'q') {
            eventName = 'connection-success-qrcode';
        } else {
            eventName = 'connection-success-manually';
        }
        
        eventParams = {
            eventName: eventName,
            params: {}
        };
        
        this.logPreviewEvent(eventParams);
    };
    
    this.logPreviewEvent = function (eventParams) {
        var socket = this.getSocket();
        if (socket) {
            socket.emit('dw:log:event', eventParams);
        }
    };
    
	this.configureWebsocket = function () {
        var socket = this.getSocket();
        
        if (socket) {
            this.initSocketCallbacks();
        } else {
            console.log("socket null");
        }
	};
    
    this.initializeSocketApp = function () {
        if (!this._initialized) {
            this.initWebsocket();
            this.configureWebsocket();
            this._initialized = true;
        }
    };
    
}

var socketApp = new SocketApp();
window.preview.socketApp = socketApp;
