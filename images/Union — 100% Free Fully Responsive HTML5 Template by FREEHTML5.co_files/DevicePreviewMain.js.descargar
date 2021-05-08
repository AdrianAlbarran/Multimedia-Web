/*
    Copyright 2015 Adobe Systems Incorporated.  All rights reserved. 

    Purpose:
        DevicePreviewMain.js is responsible for managing the preview session.
        This module communicated with Preview Service and starts the WebSocket
        App for Dw-device peer-to-peer communication.
*/

/*jslint nomen: true, plusplus: true, regexp: true */
/*global $, console, QRCode, URI*/

function Main() {
    'use strict';
    this._userId = '';
    this._sessionId = '';
    this._consumerKey = '';
    this._initState = ['UNINTIALIZED',
                       'DW_LOCAL_LOCATION_RECEIVED',
                       'DW_SERVER_LOCATION_RECEIVED',
                       'PAIRING_URL_RECEIVED',
                       'PAIRING_APP_IFRAME_INITIALIZED',
                       'USER_ID_RECEIVED',
                       'SESSION_ID_RECEIVED',
                       'DEPENDENCIES_LOADED',
                       'INIT_COMPLETE'
                      ];
    
    this._currentState = 0;
    this._appInitialized = false;
    this._isIOSDevice = false;
    
    this.setUserId = function (userId) {
        this._userId = userId;
    };

    this.getUserId = function () {
        return this._userId;
    };
 
    this.setSessionId = function (sessionId) {
        this._sessionId = sessionId;
    };

    this.getSessionId = function () {
        return this._sessionId;
    };
  
    this.setConsumerKey = function (sessionId) {
        this._consumerKey = sessionId;
    };

    this.getConsumerKey = function () {
        return this._consumerKey;
    };
    
    this.setIsIOSDevice = function (isIOSDevice) {
        this._isIOSDevice = isIOSDevice;
    };

    this.getIsIOSDevice = function () {
        return this._isIOSDevice;
    };
    
    this.initApp = function () {
        switch (this._initState[this._currentState]) {
        case 'UNINTIALIZED':
            this.retriveLocalServerAddress();
            break;
        case 'DW_LOCAL_LOCATION_RECEIVED':
            this.retrieveDwServerAddress();
            break;
        case 'DW_SERVER_LOCATION_RECEIVED':
            if (window.location.hostname === window.preview.config.DW_LOCAL_HOST) {
                this._currentState = this._currentState + 4;
                this.loadLibraries();
            } else {
                this.retrivePairingUrl();
            }
            break;
        case 'PAIRING_URL_RECEIVED':
            this.initializePairingAppFrame();
            break;
        case 'PAIRING_APP_IFRAME_INITIALIZED':
            this.requestUserId();
            break;
        case 'USER_ID_RECEIVED':
            this.requestSessionId();
            break;
        case 'SESSION_ID_RECEIVED':
            this.loadLibraries();
            break;
        case 'DEPENDENCIES_LOADED':
            window.preview.socketApp.initializeSocketApp();
            this._currentState++;
            /* falls through */
        case 'INIT_COMPLETE':
            this._appInitialized = true;
            this.setIsIOSDevice(/iPhone|iPod|iPad/.test(navigator.userAgent) && !(/Windows Phone|IEMobile/.test(navigator.userAgent)));
            
            var eventParams = {
                eventName: "login-success",
                params: {}
            };
            window.preview.socketApp.logPreviewEvent(eventParams);
            break;
        default:
            break;
        }
    };
    
    this.retrieveDwServerAddress = function () {
        var uri = new URI(window.location);
        window.preview.config.DW_SERVER_HOST = uri.hostname();
        window.preview.config.DW_SERVER_PORT = uri.port();
        this._currentState++;
        this.initApp();
    };
    
    this.retrivePairingUrl = function () {
        var self = this,
            successCallback = function (response) {
                window.preview.config.PAIRING_APP_URL = response;
                self._currentState++;
                self.initApp();
            },
            errorCallback = function () {
            };
        
        window.preview.utils.doAjaxGetRequest(
            window.location.origin,
            '/previewapp/url',
            {},
            successCallback,
            errorCallback
        );

    };
    
    this.retriveLocalServerAddress = function () {
        var self = this,
            successCallback = function (response) {
                window.preview.config.DW_LOCAL_HOST = response;
                self._currentState++;
                self.initApp();
            },
            errorCallback = function () {
            };
        
        window.preview.utils.doAjaxGetRequest(
            window.location.origin,
            '/previewapp/localurl',
            {},
            successCallback,
            errorCallback
        );

    };
    
    this.initializePairingAppFrame = function () {
        var pairingAppFrame = document.getElementById('pairingApp'),
            uriObj;
        if (pairingAppFrame) {
            uriObj = new URI(window.preview.config.PAIRING_APP_URL);
            uriObj.addSearch('sandbox', '1');
            
            pairingAppFrame.src = uriObj.toString();
            pairingAppFrame.addEventListener('load', this.pairingAppLoadedCallback);
        }
    };
    
    this.pairingAppLoadedCallback = function () {
        var self = window.preview.main,
            pairingAppFrame = document.getElementById('pairingApp');
                
        window.addEventListener('message', function (event) {
            var pairingAppUrl = new URI(window.preview.config.PAIRING_APP_URL),
                pairingRootUrl = pairingAppUrl.protocol() + '://' + pairingAppUrl.hostname();
            
            if (event.origin === pairingRootUrl) {
                console.log('message from', event.origin, event.data);
                if (event.data.key === 'user:id') {
                    self.setUserId(event.data.value);
                    self._currentState++;
                    self.initApp();
                } else if (event.data.key === 'session:id') {
                    self.setSessionId(event.data.value);
                    self._currentState++;
                    self.initApp();
                } else if (event.data === 'pairing:app:ready') {
                    self._currentState++;
                    self.initApp();
                }
            }
        }, false);
    };
    
    this.requestUserId = function () {
        var pairingAppFrame = document.getElementById('pairingApp');
        if (pairingAppFrame && pairingAppFrame.contentWindow) {
            pairingAppFrame.contentWindow.postMessage('request:user:id', window.preview.config.PAIRING_APP_URL);
        }
    };
    
    this.requestSessionId = function () {
        var pairingAppFrame = document.getElementById('pairingApp');
        if (pairingAppFrame && pairingAppFrame.contentWindow) {
            pairingAppFrame.contentWindow.postMessage('request:session:id', window.preview.config.PAIRING_APP_URL);
        }
    };
    
    this.loadLibraries = function (callback) {
        var self = this,
            doc = window.document,
            socketLib = doc.createElement('script'),
            scripts = document.head.getElementsByTagName('script'),
            clientScriptPreNode = scripts[0];
		
        socketLib.src = window.preview.config.DW_SERVER_PROTOCOL + "://" +  window.preview.config.DW_SERVER_HOST + ":" + window.preview.config.DW_SERVER_PORT + "/socket.io/socket.io.js";
		
		doc.head.insertBefore(socketLib, clientScriptPreNode);
        
        socketLib.onload = function () {
            self._currentState++;
            self.initApp();
        };
    };
}

window.preview.main = new Main();
window.preview.main.initApp();
