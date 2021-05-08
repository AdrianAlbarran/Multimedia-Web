/* Copyright 2015 Adobe Systems Incorporated.  All rights reserved. */
/*global console*/

window.addEventListener('message', function (event) {
    'use strict';
    var dwServerURL = window.preview.utils.getDwSocketAppUrl(),
        eventParams;

    if (event.origin === dwServerURL) {
        if (event.data.key === 'event') {
            if (event.data.value === 'render:success:event') {
                eventParams = {
                    eventName: 'render-success',
                    params: {}
                };
                window.preview.socketApp.logPreviewEvent(eventParams);
            }
        } else if (event.data.key === 'document:title') {
            if (event.data.value) {
                document.title = event.data.value;
            }
        }
    }
}, false);

window.onbeforeunload = function () {
    'use strict';
    window.preview.socketApp.teardown();
    return;
};
