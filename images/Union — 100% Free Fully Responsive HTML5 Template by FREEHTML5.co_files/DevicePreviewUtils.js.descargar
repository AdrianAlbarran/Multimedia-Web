/* Copyright 2015 Adobe Systems Incorporated.  All rights reserved. */
/*global $, console, URI, unescape*/
/*jslint regexp: true nomen: true, plusplus: true, bitwise: true, vars:true*/
function DevicePreviewUtils() {
    'use strict';
   
    this.createAjaxOptions = function (type, host, location, query) {
        var formedUrl = host + location,
            key,
            uri = new URI(formedUrl),
            options;
        
        
        for (key in query) {
            if (query.hasOwnProperty(key)) {
                uri.addSearch(key, query[key]);
            }
        }
        
        options = {
            'type': type,
            'url': uri.toString()
        };
        
        return options;
    };
    
    this.doAjaxGetRequest = function (host, location, query, successCallback, errorCallback) {
        var ajaxOptions = this.createAjaxOptions('GET', host, location, query);
        $.ajax(ajaxOptions).fail(function (jqXHR, textStatus, errorThrown) {
            // We got an error trying to run the work not sure what to do here
            console.log("call Failed!!  "  + textStatus);
            errorCallback.call(jqXHR, errorThrown);
        }).done(function (data, textStatus, jqXHR) {
            console.log("call succeeded!!");
            console.log(textStatus);
            successCallback.call(jqXHR, jqXHR.responseText);
        });
    };
    
    this.getAccessMethod = function () {
        var uri = new URI(window.location);
        return uri.search(true).m || '';
    };
    
    this.getDwSocketAppUrl = function () {
        return window.preview.config.DW_SERVER_PROTOCOL + '://' +
            window.preview.config.DW_SERVER_HOST +
            (window.preview.config.DW_SERVER_PORT ? ':' + window.preview.config.DW_SERVER_PORT : '');
    };
    
    this.convertPathArrayToString = function (pathArray) {
        if (!pathArray) {
            return '';
        }
        
        var path = '',
            index = 0;
        for (index = 0; index < pathArray.length; ++index) {
            var editedPath = pathArray[index];
            if (editedPath && editedPath.indexOf('DP_CLIENT_SOCKET_ID') !== -1) {
                editedPath = editedPath.replace('DP_CLIENT_SOCKET_ID', URI.encode(window.preview.socketApp.getSocket().id));
            }
            
            path += this.getDwSocketAppUrl() + editedPath;
        }
        return path;
    };
}

window.preview.utils = new DevicePreviewUtils();