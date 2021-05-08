/*
 * Copyright (c) 2015 Adobe Systems Incorporated. All rights reserved.
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

/*global $dwJQuery, DW_RefreshCSS, window */
/*jslint vars: true*/
function DeviceClientManager() {
    'use strict';
    this.inspectOverlayElemId = 'dwInspectOverlay';
    this.inspectMarginTopOverlayId = 'dwInspectMarginTopOverlay';
    this.inspectMarginBottomOverlayId = 'dwInspectMarginBottomOverlay';
    this.inspectMarginRightOverlayId = 'dwInspectMarginRightOverlay';
    this.inspectMarginLeftOverlayId = 'dwInspectMarginLeftOverlay';
    this.inspectPaddingTopOverlayId = 'dwInspectPaddingTopOverlay';
    this.inspectPaddingBottomOverlayId = 'dwInspectPaddingBottomOverlay';
    this.inspectPaddingRightOverlayId = 'dwInspectPaddingRightOverlay';
    this.inspectPaddingLeftOverlayId = 'dwInspectPaddingLeftOverlay';
    
    this.inspectTimerHandle = null;
    this.INSPECT_TIMER = 100;
    
    this.isElementVisible = function (element) {
        return element && ((element.offsetWidth > 0 && element.offsetHeight > 0) ||
            (element.style && element.style.display && element.style.display.toLowerCase() !== 'none') ||
            (element.style && element.style.visibility && element.style.visibility.toLowerCase() !== 'hidden'));
    };
    
	this.getElementRect = function (targetElement) {
		if (targetElement) {
			return targetElement.getBoundingClientRect();
		}
		return null;
	};
	
	this.isRectVisible = function (elemRect) {
		return elemRect && !(elemRect.left > window.innerWidth || elemRect.right < 0 ||
					elemRect.top > window.innerHeight || elemRect.bottom < 0);
	};
    
    this.destroyOverlays = function () {
        var inspectOverlay = document.getElementById(this.inspectOverlayElemId),
            marginTopOverlay = document.getElementById(this.inspectMarginTopOverlayId),
            marginRightOverlay = document.getElementById(this.inspectMarginRightOverlayId),
            marginBottomOverlay = document.getElementById(this.inspectMarginBottomOverlayId),
            marginLeftOverlay = document.getElementById(this.inspectMarginLeftOverlayId),
            paddingTopOverlay = document.getElementById(this.inspectPaddingTopOverlayId),
            paddingRightOverlay = document.getElementById(this.inspectPaddingRightOverlayId),
            paddingBottomOverlay = document.getElementById(this.inspectPaddingBottomOverlayId),
            paddingLeftOverlay = document.getElementById(this.inspectPaddingLeftOverlayId);

        if (inspectOverlay) {
            inspectOverlay.parentNode.removeChild(inspectOverlay);
        }
        if (marginTopOverlay) {
            marginTopOverlay.parentNode.removeChild(marginTopOverlay);
        }
        if (marginRightOverlay) {
            marginRightOverlay.parentNode.removeChild(marginRightOverlay);
        }
        if (marginBottomOverlay) {
            marginBottomOverlay.parentNode.removeChild(marginBottomOverlay);
        }
        if (marginLeftOverlay) {
            marginLeftOverlay.parentNode.removeChild(marginLeftOverlay);
        }
        if (paddingTopOverlay) {
            paddingTopOverlay.parentNode.removeChild(paddingTopOverlay);
        }
        if (paddingRightOverlay) {
            paddingRightOverlay.parentNode.removeChild(paddingRightOverlay);
        }
        if (paddingBottomOverlay) {
            paddingBottomOverlay.parentNode.removeChild(paddingBottomOverlay);
        }
        if (paddingLeftOverlay) {
            paddingLeftOverlay.parentNode.removeChild(paddingLeftOverlay);
        }
    };
 
    this.createInspectOverlay = function () {
        var inspectOverlay = document.createElement('dw-div');
        inspectOverlay.id = this.inspectOverlayElemId;
        document.getElementsByTagName('dw-container-div')[0].appendChild(inspectOverlay);
        inspectOverlay.style.display = 'block';
        inspectOverlay.style.position = 'fixed';
        inspectOverlay.style.backgroundColor = 'rgba(111, 168, 220, 0.66)';
        inspectOverlay.style.zIndex = "999999";
        return inspectOverlay;
    };
    
    this.setInspectOverlayDimensions = function (targetRect) {
        var inspectOverlay = document.getElementById(this.inspectOverlayElemId);
        if (inspectOverlay && targetRect) {
            inspectOverlay.style.height = targetRect.bottom - targetRect.top + 'px';
            inspectOverlay.style.left = targetRect.left + 'px';
            inspectOverlay.style.top = targetRect.top + 'px';
            inspectOverlay.style.width = targetRect.right - targetRect.left + 'px';
        }
    };

    this.createOrGetOverlay = function (overlyId, fillColor) {
        var marginOverlay = document.getElementById(overlyId);

        if (!marginOverlay) {
            marginOverlay = document.createElement('dw-div');
            marginOverlay.id = overlyId;
            document.getElementsByTagName('dw-container-div')[0].appendChild(marginOverlay);
            marginOverlay.style.display = 'block';
            marginOverlay.style.position = 'fixed';
            marginOverlay.style.backgroundColor = fillColor;
            marginOverlay.style.zIndex = "999999";
        }
        return marginOverlay;
    };
    
    this.positionMarginOverlay = function (element, inspectOverlay, marginProp) {
        var style = element.currentStyle || window.getComputedStyle(element),
            marginValue,
            marginFill = 'rgba(246, 178, 107, 0.66)',
            marginOverlay,
            marginTopOverlay,
            marginRightOverlay,
            marginBottomOverlay,
            targetRect = null;
            

        if (marginProp === 'top') {
            marginValue = style.marginTop;
            marginOverlay = this.createOrGetOverlay(this.inspectMarginTopOverlayId, marginFill);
        } else if (marginProp === 'right') {
            marginValue = style.marginRight;
            marginOverlay = this.createOrGetOverlay(this.inspectMarginRightOverlayId, marginFill);
        } else if (marginProp === 'bottom') {
            marginValue = style.marginBottom;
            marginOverlay = this.createOrGetOverlay(this.inspectMarginBottomOverlayId, marginFill);
        } else {
            marginValue = style.marginLeft;
            marginOverlay = this.createOrGetOverlay(this.inspectMarginLeftOverlayId, marginFill);
        }

        if (parseFloat(marginValue) > 0) {
            if (marginProp === 'top') {
                marginOverlay.style.left = inspectOverlay.style.left;
                marginOverlay.style.width = inspectOverlay.style.width;
                marginOverlay.style.height = marginValue;
                marginOverlay.style.top = parseFloat(inspectOverlay.style.top) - parseFloat(marginValue) + 'px';
            } else if (marginProp === 'right') {
                marginTopOverlay = this.createOrGetOverlay(this.inspectMarginTopOverlayId, marginFill);
                marginOverlay.style.left = parseFloat(inspectOverlay.style.left) + parseFloat(inspectOverlay.style.width) + 'px';
                marginOverlay.style.width = marginValue;
                if (marginTopOverlay && marginTopOverlay.style && marginTopOverlay.style.visibility === "visible") {
                    marginOverlay.style.top = parseFloat(inspectOverlay.style.top) - parseFloat(marginTopOverlay.style.height) + 'px';
                    marginOverlay.style.height = parseFloat(inspectOverlay.style.height) + parseFloat(marginTopOverlay.style.height) + 'px';
                } else {
                    marginOverlay.style.top = inspectOverlay.style.top;
                    marginOverlay.style.height = inspectOverlay.style.height;
                }
            } else if (marginProp === 'bottom') {
                marginRightOverlay = this.createOrGetOverlay(this.inspectMarginRightOverlayId, marginFill);
                marginOverlay.style.left = inspectOverlay.style.left;
                if (marginRightOverlay && marginRightOverlay.style && marginRightOverlay.style.visibility === "visible") {
                    marginOverlay.style.width = parseFloat(inspectOverlay.style.width) + parseFloat(marginRightOverlay.style.width) + 'px';
                } else {
                    marginOverlay.style.width = inspectOverlay.style.width;
                }
                marginOverlay.style.height = marginValue;
                marginOverlay.style.top = parseFloat(inspectOverlay.style.top) + parseFloat(inspectOverlay.style.height) + 'px';
            } else {
                marginBottomOverlay = this.createOrGetOverlay(this.inspectMarginBottomOverlayId, marginFill);
                marginTopOverlay = this.createOrGetOverlay(this.inspectMarginTopOverlayId, marginFill);
                marginOverlay.style.left = parseFloat(inspectOverlay.style.left) - parseFloat(marginValue) + 'px';
                marginOverlay.style.width = marginValue;
                if (marginBottomOverlay && marginBottomOverlay.style && marginBottomOverlay.style.visibility === "visible") {
                    marginOverlay.style.height = parseFloat(inspectOverlay.style.height) + parseFloat(marginBottomOverlay.style.height) + 'px';
                } else {
                    marginOverlay.style.height = inspectOverlay.style.height;
                }
                if (marginTopOverlay && marginTopOverlay.style && marginTopOverlay.style.visibility === "visible") {
                    marginOverlay.style.top = parseFloat(inspectOverlay.style.top) - parseFloat(marginTopOverlay.style.height) + 'px';
                    marginOverlay.style.height = parseFloat(marginOverlay.style.height) + parseFloat(marginTopOverlay.style.height) + 'px';
                } else {
                    marginOverlay.style.top = inspectOverlay.style.top;
                }
            }
            marginOverlay.style.visibility = "visible";
            if (marginProp === 'top' && (targetRect = this.getElementRect(marginOverlay)) && !this.isRectVisible(targetRect)) {
                marginOverlay.scrollIntoView();
                marginOverlay.style.left = inspectOverlay.style.left;
                marginOverlay.style.top = parseFloat(inspectOverlay.style.top) - parseFloat(marginValue) + 'px';
                this.setInspectOverlayDimensions(this.getElementRect(element));
            }
        } else {
            marginOverlay.style.visibility = "hidden";
        }
    };

    this.positionPaddingOverlay = function (element, inspectOverlay, paddingProp) {
        var style = element.currentStyle || window.getComputedStyle(element),
            paddingValue,
            paddingFill = 'rgba(147, 196, 125, 0.55)',
            paddingOverlay;

        if (paddingProp === 'top') {
            paddingValue = style.paddingTop;
            paddingOverlay = this.createOrGetOverlay(this.inspectPaddingTopOverlayId, paddingFill);
        } else if (paddingProp === 'right') {
            paddingValue = style.paddingRight;
            paddingOverlay = this.createOrGetOverlay(this.inspectPaddingRightOverlayId, paddingFill);
        } else if (paddingProp === 'bottom') {
            paddingValue = style.paddingBottom;
            paddingOverlay = this.createOrGetOverlay(this.inspectPaddingBottomOverlayId, paddingFill);
        } else {
            paddingValue = style.paddingLeft;
            paddingOverlay = this.createOrGetOverlay(this.inspectPaddingLeftOverlayId, paddingFill);
        }

        if (parseFloat(paddingValue) > 0) {
            if (paddingProp === 'top') {
                paddingOverlay.style.left = inspectOverlay.style.left;
                paddingOverlay.style.width = inspectOverlay.style.width;
                paddingOverlay.style.height = paddingValue;
                paddingOverlay.style.top = inspectOverlay.style.top;
            } else if (paddingProp === 'right') {
                paddingOverlay.style.left = parseFloat(inspectOverlay.style.left) + parseFloat(inspectOverlay.style.width) - parseFloat(paddingValue) + 'px';
                paddingOverlay.style.width = paddingValue;
                paddingOverlay.style.height = inspectOverlay.style.height;
                paddingOverlay.style.top = inspectOverlay.style.top;
            } else if (paddingProp === 'bottom') {
                paddingOverlay.style.left = inspectOverlay.style.left;
                paddingOverlay.style.width = inspectOverlay.style.width;
                paddingOverlay.style.height = paddingValue;
                paddingOverlay.style.top = parseFloat(inspectOverlay.style.top) + parseFloat(inspectOverlay.style.height) - parseFloat(paddingValue) + 'px';
            } else {
                paddingOverlay.style.left = inspectOverlay.style.left;
                paddingOverlay.style.width = paddingValue;
                paddingOverlay.style.height = inspectOverlay.style.height;
                paddingOverlay.style.top = inspectOverlay.style.top;
            }
            paddingOverlay.style.visibility = "visible";
        } else {
            paddingOverlay.style.visibility = "hidden";
        }
    };

    this.createAndPositionMarginOverlays = function (element, inspectOverlay) {
        this.positionMarginOverlay(element, inspectOverlay, 'top');
        this.positionMarginOverlay(element, inspectOverlay, 'right');
        this.positionMarginOverlay(element, inspectOverlay, 'bottom');
        this.positionMarginOverlay(element, inspectOverlay, 'left');
    };

    this.createAndPositionPaddingOverlays = function (element, inspectOverlay) {
        this.positionPaddingOverlay(element, inspectOverlay, 'top');
        this.positionPaddingOverlay(element, inspectOverlay, 'right');
        this.positionPaddingOverlay(element, inspectOverlay, 'bottom');
        this.positionPaddingOverlay(element, inspectOverlay, 'left');
    };
    
    this.hideAllOverlays = function () {
        this.createOrGetOverlay(this.inspectPaddingTopOverlayId, '').style.visibility = "hidden";
        this.createOrGetOverlay(this.inspectPaddingBottomOverlayId, '').style.visibility = "hidden";
        this.createOrGetOverlay(this.inspectPaddingLeftOverlayId, '').style.visibility = "hidden";
        this.createOrGetOverlay(this.inspectPaddingRightOverlayId, '').style.visibility = "hidden";
        this.createOrGetOverlay(this.inspectMarginBottomOverlayId, '').style.visibility = "hidden";
        this.createOrGetOverlay(this.inspectMarginLeftOverlayId, '').style.visibility = "hidden";
        this.createOrGetOverlay(this.inspectMarginRightOverlayId, '').style.visibility = "hidden";
        this.createOrGetOverlay(this.inspectMarginTopOverlayId, '').style.visibility = "hidden";
        var inspectOverlay = document.getElementById(this.inspectOverlayElemId);
        if (inspectOverlay) {
            inspectOverlay.style.visibility = "hidden";
        }
    };
    
    this.doInspect = function (liveEditId, selectorString) {
        var self = this;
        
        if (this.inspectTimerHandle) {
            window.clearTimeout(this.inspectTimerHandle);
        }
        
        this.inspectTimerHandle = window.setTimeout(function () {
            self.doInspectImpl(liveEditId, selectorString);
        }, this.INSPECT_TIMER);
    };
    
    this.doInspectImpl = function (liveEditId, selectorString) {
        var inspectOverlay,
            elementList = document.querySelectorAll("[data_liveedit_tagid='" + liveEditId + "']"),
            element = null,
            targetRect = null;
        
        if (elementList.length !== 0) {
            element = elementList[0];
        } else if (selectorString) {
            elementList = document.querySelectorAll(selectorString);
            if (elementList.length !== 0) {
                element = elementList[0];
            }
        }
        
        if (!(element && this.isElementVisible(element))) {
            this.hideAllOverlays();
            return;
        }
		
        inspectOverlay = document.getElementById(this.inspectOverlayElemId);
        if (!inspectOverlay) {
            inspectOverlay = this.createInspectOverlay();
        }
        
        if ((targetRect = this.getElementRect(element)) && !this.isRectVisible(targetRect)) {
            element.scrollIntoView();
			targetRect = this.getElementRect(element);
        }
		
        inspectOverlay.style.visibility = "visible";
        this.setInspectOverlayDimensions(targetRect);
        this.createAndPositionMarginOverlays(element, inspectOverlay);
        this.createAndPositionPaddingOverlays(element, inspectOverlay);
    };
    
    this.doScrollElementIntoView = function (liveId, selectorString) {
        var elementList = document.querySelectorAll("[data_liveedit_tagid='" + liveId + "']"),
            element,
            targetRect;
        
        if (elementList.length !== 0) {
            element = elementList[0];
        } else if (selectorString) {
            elementList = document.querySelectorAll(selectorString);
            if (elementList.length !== 0) {
                element = elementList[0];
            }
        }
        
        if (element && this.isElementVisible(element) && !this.isRectVisible(this.getElementRect(element))) {
            element.scrollIntoView();
        }
    };
}


window.addEventListener("message", function (e) {
    'use strict';
    if (e.data.event === 'scroll') {
        var xperc = e.data.xperc,
            yperc = e.data.yperc,
            xoffset = (xperc / 100) * (document.body.scrollWidth - window.innerWidth),
            yoffset = (yperc / 100) * (document.body.scrollHeight - window.innerHeight);

        window.scrollTo(xoffset, yoffset);
        
    } else if (e.data.event === 'inspectOff') {
        
        window.preview.deviceClientManager.destroyOverlays();
    
    } else if (e.data.event === 'inspect') {

        window.preview.deviceClientManager.doInspect(e.data.liveEditId, e.data.selectorString);
        
    } else if (e.data.event === 'scrollElement') {

        window.preview.deviceClientManager.doScrollElementIntoView(e.data.liveEditId, e.data.selectorString);
        
    } else if (e.data.event === 'partialRefreshDeleteRule') {
        
        DW_RefreshCSS.deleteRule(e.data.rule.mediaText, e.data.rule.cssText, e.data.rule.encodedPath);
    
    } else if (e.data.event === 'partialRefreshInsertRule') {
    
        DW_RefreshCSS.insertRule(e.data.rule.mediaText, e.data.rule.cssText, e.data.rule.encodedPath);
    
    } else if (e.data.event === 'partialRefreshHTML') {
        
        window.DW_LiveEdit_DoEdit(e.data.args.startOffset, e.data.args.endOffset, e.data.args.editString, false, e.data.args.fullRefreshIfFailed);
    
    }
}, false);

window.onload = function () {
    'use strict';
    
    window.parent.postMessage('render:success:event', '*');
    
    if (window.DevicePreview_RenderSuccess) {
        window.parent.postMessage({
            'key': 'event',
            'value': 'render:success:event'
        }, '*');
    }
    
    window.parent.postMessage({
        'key': 'document:title',
        'value': document.title
    }, '*');
    
    return true;
};

window.addEventListener("resize", function () {
    'use strict';
    if (window.preview && window.preview.deviceClientManager) {
        window.preview.deviceClientManager.destroyOverlays();
    }
    return true;
});
                        
window.preview = window.preview || {};
window.preview.deviceClientManager = window.preview.deviceClientManager || new DeviceClientManager();

