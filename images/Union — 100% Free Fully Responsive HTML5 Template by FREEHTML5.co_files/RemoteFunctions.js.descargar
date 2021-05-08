/*
 * Copyright (c) 2012 Adobe Systems Incorporated. All rights reserved.
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
/*jslint vars: true, plusplus: true, browser: true, nomen: true, indent: 4, forin: true, maxerr: 50, regexp: true, white: false */
/*global define, $dwJQuery, window, navigator, Node, console,DW_LIVEEDIT_CONSTANTS, DW_LiveEdit_IDManagerInstance, DW_LiveEdit_ReplacedDOM */
/*theseus instrument: false */
/**
 * RemoteFunctions define the functions to be executed in the browser. This
 * modules should define a single function that returns an object of all
 * exported functions.
 */
function DW_RemoteFunctions_Impl() {
    "use strict";

    var DW_CUSTOM_DIVTAG = 'dw-container-div';
    var DW_LIVEEDIT_ID = 'data_liveedit_tagid';
    var lastKeepAliveTime = Date.now();

    /**
     * @type {DOMEditHandler}
     */
    var _editHandler;

    /**
     * Constructor
     * @param {Document} htmlDocument
     */
    function DOMEditHandler(htmlDocument) {
        this.htmlDocument = htmlDocument;
        this.rememberedNodes = null;
        this.entityParseParent = htmlDocument.createElement("div");
        this.hasInstruction = false;
    }

    function applyDOMEditsHasInstruction() {
        return _editHandler.hasInstruction;
    }

    function resetApplyDOMEditsHasInstruction() {
        _editHandler.hasInstruction = false;
    }

    /**
     * @private
     * Find the first matching element with the specified data_liveedit_tagid
     * @param {string} id
     * @return {Element}
     */
    DOMEditHandler.prototype._queryBracketsID = function (id) {
        if (!id) {
            return null;
        }

        if (this.rememberedNodes && this.rememberedNodes[id]) {
            return this.rememberedNodes[id];
        }

        id = DW_LiveEdit_IDManagerInstance.DW_LiveEdit_ID2dwIDMap[id];
        var results = this.htmlDocument.querySelectorAll("[" + DW_LIVEEDIT_ID + "='" + id + "']");
        return results && results[0];
    };

    /**
     * @private
     * Insert a new child element
     * @param {Element} targetElement Parent element already in the document
     * @param {Element} childElement New child element
     * @param {Object} edit
     */
    DOMEditHandler.prototype._insertChildNode = function (targetElement, childElement, edit) {
        var before = this._queryBracketsID(edit.beforeID),
            after = this._queryBracketsID(edit.afterID);

        if (edit.firstChild) {
            before = targetElement.firstChild;
        } else if (edit.lastChild) {
            after = targetElement.lastChild;
        }

        if (before) {
            if (before.parentNode === targetElement) {
                targetElement.insertBefore(childElement, before);
            } else {
                return false;
            }
        } else if (after && (after !== targetElement.lastChild)) {
            if (after.nextSibling.parentNode === targetElement) {
                targetElement.insertBefore(childElement, after.nextSibling);
            } else {
                return false;
            }
        } else {
            targetElement.appendChild(childElement);
        }
    };

    /**
     * @private
     * Insert a new child Text element
     * @param {Element} targetElement Parent element already in the document
     * @param {Element} Text Content
     * @param {Object} edit
     */
    DOMEditHandler.prototype._insertChildTextNode = function (targetElement, childText, edit) {
        var before = this._queryBracketsID(edit.beforeID),
            after = this._queryBracketsID(edit.afterID);

        if (edit.firstChild) {
            before = targetElement.firstChild;
        } else if (edit.lastChild) {
            after = targetElement.lastChild;
        }

        if (before) {
            if (before.parentNode === targetElement) {
                before.insertAdjacentHTML("beforeBegin", childText);
            } else {
                return false;
            }
        } else if (after && (after !== targetElement.lastChild)) {
            if (after.nextSibling.parentNode === targetElement) {
                after.nextSibling.insertAdjacentHTML("beforeBegin", childText);
            } else {
                return false;
            }
        } else {
            targetElement.insertAdjacentHTML("beforeEnd", childText);
        }
    };

    /**
     * @private
     * Given a string containing encoded entity references, returns the string with the entities decoded.
     * @param {string} text The text to parse.
     * @return {string} The decoded text.
     */
    DOMEditHandler.prototype._parseEntities = function (text) {
        // Kind of a hack: just set the innerHTML of a div to the text, which will parse the entities, then
        // read the content out.
        var result;
        this.entityParseParent.innerHTML = text;
        result = this.entityParseParent.textContent;
        this.entityParseParent.textContent = "";
        return result;
    };

    /**
     * @private
     * Given a DW ID, see if there is conflicting LiveEdit ID, update the required map with newly generated ID
     * @param {string} DWID {string} LiveEdit ID used in partial Refresh
     * @return nothing
     */
    DOMEditHandler.prototype._updateLiveEditID = function (dwID, tagID) {
        //this is a tricky part, we are adding a new element
        //it is possible that it will share dwID with another element which is bound to be deleted later
        //we will change the dwID of the latter to something unique and update out map
        //we can look at our temp map for the required ID
        //we assume that the dwID has not been set on the new element
        var mapRef = window.DW_LiveEdit_IDManagerInstance.DW_LiveEdit_ID2dwIDMap;
        if (dwID) {
            var results = this.htmlDocument.querySelectorAll("[" + DW_LIVEEDIT_ID + "='" + dwID + "']");
            var elem = results && results[0];
            if (elem) {
                //there is already an element with the same ID
                //Most probably this element is bound for deletion
                //we will now replace its dwID to something different
				var id = null;
                for (id in mapRef) {
                    if (mapRef.hasOwnProperty(id) && mapRef[id] === dwID) {
                        var newValue = "conflictTag" + id;
                        elem.setAttribute(DW_LIVEEDIT_ID, newValue);
                        mapRef[id] = newValue;
                        break;
                    }
                }
            }
            mapRef[tagID] = dwID;
        }
    };

    /**
     * @private
     * Given an SimpleNode remove reference from DW_LiveEdit_ID2dwIDMap of it and its children
     * @param {element} element to be deleted;
     * @return nothing
     */
    DOMEditHandler.prototype._removeLiveEditID = function (simpleElem) {
        var mapRef = window.DW_LiveEdit_IDManagerInstance.DW_LiveEdit_ID2dwIDMap;
        if (simpleElem && simpleElem.isElement() && mapRef) {
            if (simpleElem.tagID && mapRef[simpleElem.tagID]) {
                delete mapRef[simpleElem.tagID];
            }
            if (simpleElem.children) {
				var index = 0;
                for (index = 0; index < simpleElem.children.length; ++index) {
                    this._removeLiveEditID(simpleElem.children[index]);
                }
            }
        }
    };

    /**
     * @private
     * Given an SimpleNode adds reference from DW_LiveEdit_ID2dwIDMap of it and its children
     * @param {element} element to be added;
     * @return nothing
     */
    DOMEditHandler.prototype._addLiveEditID = function (simpleElem) {
        var mapRef = window.DW_LiveEdit_IDManagerInstance.DW_LiveEdit_ID2dwIDMap;
        if (simpleElem && simpleElem.isElement() && mapRef) {
            if (simpleElem.tagID && !mapRef[simpleElem.tagID] && simpleElem.attributes && simpleElem.attributes[DW_LIVEEDIT_ID]) {
                mapRef[simpleElem.tagID] = simpleElem.attributes[DW_LIVEEDIT_ID];
            }
            if (simpleElem.children) {
				var index = 0;
                for (index = 0; index < simpleElem.children.length; ++index) {
                    this._addLiveEditID(simpleElem.children[index]);
                }
            }
        }
    };

    /**
     * @private
     * @param {Node} node
     * @return {boolean} true if node expects its content to be raw text (not parsed for entities) according to the HTML5 spec.
     */
    function _isRawTextNode(node) {
        return (node.nodeType === Node.ELEMENT_NODE && /script|style|noscript|noframes|noembed|iframe|xmp/i.test(node.tagName));
    }

    DOMEditHandler.prototype._checkForDynamicImpl = function (id, elem) {
        if (DW_LiveEdit_ReplacedDOM && DW_LiveEdit_ReplacedDOM.nodeMap) {
            var nMap = DW_LiveEdit_ReplacedDOM.nodeMap;
            var simpleNode = nMap[id];
            if (simpleNode && elem) {
			
				if (simpleNode && simpleNode.hasInstruction) {
					return false; // If our edit has server tags, do not proceed
				}
				
                var attrName = "";
                var nodeVal = "";
                var elemVal = "";
				//first check if all attr in node is there in element
                var nodeAttrs = simpleNode.attributes;
				if (nodeAttrs) {
					for (attrName in nodeAttrs) {
						if (nodeAttrs.hasOwnProperty(attrName)) {
							nodeVal = nodeAttrs[attrName];
							elemVal = elem.getAttribute(attrName);
							if ((nodeVal !== elemVal) && !(!!nodeVal === false && !!elemVal === false)) {
								return false;
							}
						}
					}
				}

				//now check for all attr in elem is there in node
                var index = 0;
				var elemAttrs = elem.attributes;
				if (elemAttrs) {
					for (index = 0; index < elemAttrs.length; ++index) {
						attrName = elemAttrs[index].name;
						nodeVal = nodeAttrs ? nodeAttrs[attrName] : null;
						elemVal = elemAttrs[index].value;
						if ((nodeVal !== elemVal) && !(!!nodeVal === false && !!elemVal === false)) {
							return false;
						}
					}
				}

				//check if all childelements have dw-id && concat all text inside the element
				var elemChildren = elem.childNodes;
				var elemText = "";
                var childElement = null;
				for (index = 0; index < elemChildren.length; ++index) {
					childElement = elemChildren[index];
					if (childElement) {
						switch (childElement.nodeType) {
						case Node.ELEMENT_NODE:
							if (childElement.tagName.toLowerCase() !== "dw-container-div" && !childElement.getAttribute(DW_LIVEEDIT_ID)) {
								return false;
							}
							break;
						case Node.TEXT_NODE:
							elemText += childElement.textContent;
							break;
						default:
							break;
						}
					}
				}

				//concat all the text in the simpleNode
				var nodeChildren = simpleNode.children;
				var nodeText = "";
                childElement = null;
				if (nodeChildren) {
					for (index = 0; index < nodeChildren.length; ++index) {
						childElement = nodeChildren[index];
						if (childElement && !childElement.isElement()) {
							nodeText += childElement.content;
						}
					}
				}

				//need to convert text for HTML to regular text for formatting except for script and style tag
				if (simpleNode.tag && simpleNode.tag.toLowerCase() !== "script" && simpleNode.tag.toLowerCase() !== "style") {
					var tempElem = document.createElement('div');
					tempElem.innerHTML = nodeText;
					nodeText = tempElem.textContent;
				}

				//verify the text inside the simple node is same as the text in the HTMLElement
				if (elemText.replace(/\s/g, "").toLowerCase() !== nodeText.replace(/\s/g, "").toLowerCase()) {
					return false;
				}
				return true;
			}
        }
		return false;
    };

    /**
     * @private
     * @param array of edits
     * @return {boolean} true if the attribute of the edits are consistent, false other wise
     */
    DOMEditHandler.prototype._checkForDynamicContent = function (edits) {
        if (!edits) {
            return true;
		}
        var verifiedMap = {};
		var index = 0;
        for (index = 0; index < edits.length; ++index) {
            var edit = edits[index];
            var targetID = edit.type.match(/textReplace|textDelete|textInsert|elementInsert|elementMove/) ? edit.parentID : edit.tagID;
            var targetElement = this._queryBracketsID(targetID);
            if (targetElement && !verifiedMap[targetID]) {
				if (this._checkForDynamicImpl(targetID, targetElement)) {
					verifiedMap[targetID] = true;
				} else {
					return false;
				}
				
				//disable partial refresh if elementDelete edit for canvas tag only.
				//earlier, partial refresh was also disabled for elementDelete or textReplace edit for script tag.
				if (edit.type === "elementDelete" && (targetElement.tagName.toLowerCase() === "canvas")) {
					return false;
				}
                
            }
        }

        return true;
    };

    /**
     * @private
     * Replace a range of text and comment nodes with an optional new text node
     * @param {Element} targetElement
     * @param {Object} edit
     */
    DOMEditHandler.prototype._textReplace = function (targetElement, edit) {
        function prevIgnoringHighlights(node) {
            node = node.previousSibling;
            while (node && node.nodeType === Node.ELEMENT_NODE && node.getAttribute(DW_LIVEEDIT_ID) === null) {
                end = node;
                node = node.previousSibling;
            }
            return node;
        }

        function nextIgnoringHighlights(node) {
            node = node.nextSibling;
            while (node && node.nodeType === Node.ELEMENT_NODE && node.getAttribute(DW_LIVEEDIT_ID) === null) {
                start = node;
                node = node.nextSibling;
            }
            return node;
        }

        function lastChildIgnoringHighlights(node) {
            node = (node.childNodes.length ? node.childNodes.item(node.childNodes.length - 1) : null);
            if (node && node.nodeType === Node.ELEMENT_NODE && node.getAttribute(DW_LIVEEDIT_ID) === null) {
                end = node;
                node = prevIgnoringHighlights(node);
            }
            return node;
        }

        var start = (edit.afterID) ? this._queryBracketsID(edit.afterID) : null,
            startMissing = edit.afterID && !start,
            end = (edit.beforeID) ? this._queryBracketsID(edit.beforeID) : null,
            endMissing = edit.beforeID && !end,
            moveNext = start && nextIgnoringHighlights(start),
            current = moveNext || (end && prevIgnoringHighlights(end)) || lastChildIgnoringHighlights(targetElement),
            next,
            lastRemovedWasText,
            isText;

        var textContents = (edit.content !== undefined) ? edit.content : null;

        // remove all nodes inside the range
        while (current && (current !== end)) {
            isText = current.nodeType === Node.TEXT_NODE || current.nodeType === Node.COMMENT_NODE;

            // if start is defined, delete following text nodes
            // if start is not defined, delete preceding text nodes
            next = (moveNext) ? nextIgnoringHighlights(current) : prevIgnoringHighlights(current);

            // only delete up to the nearest element.
            // if the start/end tag was deleted in a prior edit, stop removing
            // nodes when we hit adjacent text nodes
            if ((current.nodeType === Node.ELEMENT_NODE) ||
                    ((startMissing || endMissing) && (isText && lastRemovedWasText))) {
                break;
            } else {
                lastRemovedWasText = isText;
                current.parentNode.removeChild(current);
                current = next;
            }
        }

        if (textContents) {
            // OK to use nextSibling here (not nextIgnoringHighlights) because we do literally
            // want to insert immediately after the start tag.
            if (start && start.nextSibling) {
                if (start.nextSibling.parentNode === targetElement) {
                    start.nextSibling.insertAdjacentHTML("beforeBegin", textContents);
                } else {
                    return false;
                }
            } else if (end) {
                if (end.parentNode === targetElement) {
                    end.insertAdjacentHTML("beforeBegin", textContents);
                } else {
                    return false;
                }
            } else {
                targetElement.insertAdjacentHTML("beforeEnd", textContents);
            }
        }
    };

    /**
     * @private
     * Apply an array of DOM edits to the document
     * @param {Array.<Object>} edits
     */
    DOMEditHandler.prototype.apply = function (edits) {
        var targetID,
            targetElement,
            childElement,
            self = this;

        if (!this._checkForDynamicContent(edits)) {
            return false;
        }

        this.rememberedNodes = {};
        this.retVal = true;
        this.dwCustomElement = null;

        $dwJQuery.each(edits, function (index, edit) {
            var editIsSpecialTag = edit.type === "elementInsert" && (edit.tag === "html" || edit.tag === "head" || edit.tag === "body");

            if (edit.type === "rememberNodes") {
                edit.tagIDs.forEach(function (tagID) {
                    var node = self._queryBracketsID(tagID);
                    self.rememberedNodes[tagID] = node;
                    node.parentNode.removeChild(node);
                });
                return true;
            }

            targetID = edit.type.match(/textReplace|textDelete|textInsert|elementInsert|elementMove/) ? edit.parentID : edit.tagID;
            targetElement = self._queryBracketsID(targetID);

            var simpleDOMNode = null;
            if (edit.type.match(/textReplace|textDelete|textInsert/)) {
                simpleDOMNode = window.DW_LiveEdit_CurrentSimpleDOM.nodeMap[edit.parentID];
            } else if (edit.type.match(/elementInsert|elementDelete/)) {
                simpleDOMNode = window.DW_LiveEdit_CurrentSimpleDOM.nodeMap[edit.tagID];
            }

            if (!targetElement && !editIsSpecialTag) {
                console.error(DW_LIVEEDIT_ID + "=" + targetID + " not found");
                self.retVal = false;
                return false;
            }

            var funcReturn = null;

            switch (edit.type) {
            case "attrChange":
            case "attrAdd":
                if (edit.attribute.toLowerCase() === DW_LIVEEDIT_ID) {
                    self._updateLiveEditID(self._parseEntities(edit.value), targetID);
                }

                targetElement.setAttribute(edit.attribute, self._parseEntities(edit.value));
                if (targetElement.tagName.toLowerCase() === "img") {
                    if (edit.attribute.toLowerCase() === "src") {
                        if (!(edit.value && edit.value.length)) {
                            targetElement.setAttribute("src", "temp value to clear cached drawing of image in CEF");
                        }
                    }
                }

                //for input type color, setting the 'value' with setAttribute doesn't work. SO, setting it through this way
                if (targetElement.tagName.toLowerCase() === "input" && edit.attribute.toLowerCase() === "value") {
                    var attrVal = targetElement.getAttribute('type');
                    if (attrVal && attrVal.toLowerCase() === "color") {
                        targetElement.value = self._parseEntities(edit.value);
                    }
                }
                break;
            case "attrDelete":
                if (targetElement.tagName.toLowerCase() === "img") {
                    if (edit.attribute.toLowerCase() === "src") {
                        if (!(edit.value && edit.value.length)) {
                            targetElement.setAttribute("src", "temp value to clear cached drawing of image in CEF");
                        }
                    }
                }
                targetElement.removeAttribute(edit.attribute);
                break;
            case "elementDelete":
                if (!self.dwCustomElement && (targetElement === self.htmlDocument.body || targetElement.parentNode === self.htmlDocument.body)) {
                    self.dwCustomElement = self.htmlDocument.getElementsByTagName(DW_CUSTOM_DIVTAG)[0]; // if we are deleting the body keep our custom div tag and insert it again later
                    self.htmlDocument.body.removeChild(self.dwCustomElement);
                }

                if (DW_LiveEdit_ReplacedDOM && DW_LiveEdit_ReplacedDOM.nodeMap && edit.tagID) {
                    var elem = DW_LiveEdit_ReplacedDOM.nodeMap[edit.tagID];
                    self._removeLiveEditID(elem);
                }

                //targetElement.remove(); does not work on elements like select
                if (!(targetElement.parentNode && targetElement.parentNode.removeChild(targetElement))) {
                    self.retVal = false;
                }
                break;
            case "elementInsert":
                childElement = null;

				if (!self.dwCustomElement && (targetElement === self.htmlDocument.body || (simpleDOMNode && simpleDOMNode.tag === "body"))) {
                    self.dwCustomElement = self.htmlDocument.getElementsByTagName(DW_CUSTOM_DIVTAG)[0]; // if we are inserting the body / body's child keep our custom div tag and insert it again later
                    self.htmlDocument.body.removeChild(self.dwCustomElement);
                }

                if (editIsSpecialTag) {
                    // If we already have one of these elements (which we should), then
                    // just copy the attributes and set the ID.
                    childElement = self.htmlDocument[edit.tag === "html" ? "documentElement" : edit.tag];
                    if (!childElement) {
                        // Treat this as a normal insertion.
                        editIsSpecialTag = false;
                    }
                }
                if (!editIsSpecialTag) {
                    childElement = self.htmlDocument.createElement(edit.tag);
                }

                Object.keys(edit.attributes).forEach(function (attr) {
                    childElement.setAttribute(attr, self._parseEntities(edit.attributes[attr]));
                });

                self._updateLiveEditID(childElement.getAttribute(DW_LIVEEDIT_ID), edit.tagID);

                if (!editIsSpecialTag) {
                    funcReturn = self._insertChildNode(targetElement, childElement, edit);
                }

                if (childElement.tagName.toLowerCase() === "script" && childElement.getAttribute('src')) {
                    $dwJQuery.getScript(childElement.getAttribute('src'));
                }
                break;
            case "elementMove":
                childElement = self._queryBracketsID(edit.tagID);

                if (childElement && DW_LiveEdit_ReplacedDOM && DW_LiveEdit_ReplacedDOM.nodeMap && edit.tagID) {
                    var curElem = DW_LiveEdit_ReplacedDOM.nodeMap[edit.tagID];
                    self._addLiveEditID(curElem);
                }

                funcReturn = self._insertChildNode(targetElement, childElement, edit);
                break;
            case "textInsert":
                if (edit.content !== undefined && targetElement.tagName.toLowerCase() !== 'html') {
                    funcReturn = self._insertChildTextNode(targetElement, edit.content, edit);
                }
                break;
            case "textReplace":
            case "textDelete":
				if (targetElement.tagName.toLowerCase() !== 'html') {
					funcReturn = self._textReplace(targetElement, edit);
				}
                break;
            }
            if (funcReturn === false) {
                self.retVal = false;
            }
            return self.retVal;
        });

        if (this.dwCustomElement && this.retVal) {
            this.htmlDocument.body.appendChild(this.dwCustomElement); // if we plucked out DW specific tag, place them back in the body
        }

        this.rememberedNodes = {};
        return this.retVal;
    };

    function applyDOMEdits(edits) {
        return _editHandler.apply(edits);
    }

    // init
    _editHandler = new DOMEditHandler(window.document);


    return {
        "DOMEditHandler": DOMEditHandler,
        "applyDOMEdits": applyDOMEdits,
        "applyDOMEditsHasInstruction": applyDOMEditsHasInstruction,
        "resetApplyDOMEditsHasInstruction": resetApplyDOMEditsHasInstruction
    };
}

var DW_RemoteFunctions_Instance = null;

function DW_RemoteFunctions() {
    "use strict";
    if (!DW_RemoteFunctions_Instance) {
        DW_RemoteFunctions_Instance = DW_RemoteFunctions_Impl();
    }
    return DW_RemoteFunctions_Instance;
}
