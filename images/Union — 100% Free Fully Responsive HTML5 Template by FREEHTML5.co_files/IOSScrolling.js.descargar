/*
 * Copyright (c) 2013 Adobe Systems Incorporated. All rights reserved.
 *
 * Permission is hereby granted, free of charge, to any person obtaining a
 * copy of this software and associated documentation files (the "Software"),
 * to deal in the Software without restriction, including without limitation
 * the rights to use, copy, modify, merge, publish, distribute, sublicense,
 * and/or sell copies of the Software, and to permit persons to whom the
 * Software is furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
 * DEALINGS IN THE SOFTWARE.
 *
 */
/*jslint vars:true*/
window.dwHandleIOSScrolling = function () {
    'use strict';
    if (/iPhone|iPod|iPad/.test(navigator.userAgent) && !(/Windows Phone|IEMobile/.test(navigator.userAgent))) {
        var containers = document.getElementsByTagName('dw-container-div');
        if (containers && containers.length > 0) {
            var styleElem = document.createElement('style');
            styleElem.innerHTML = "html,body{position:fixed;top:0;left:0;right:0;bottom:0;overflow:scroll;-webkit-overflow-scrolling:touch;}";
            containers[0].insertBefore(styleElem, containers[0].firstChild);
        }
    }
};

window.dwHandleIOSScrolling();