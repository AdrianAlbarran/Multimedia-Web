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
/*jslint vars: true, plusplus: true, devel: true, nomen: true, indent: 4, maxerr: 50, continue: true, newcap:true, regexp:true, newcap:true*/
/*global DW_HTMLSimpleDOM, window, DW_LIVEEDIT_CONSTANTS, $dwJQuery, DW_HTMLDOMDiff, DW_RemoteFunctions, document, CSSRule, Node*/
/*Global Vars*/
var DW_LiveEdit_UniqueID = 'data_liveedit_tagid';
var DW_LiveEdit_IDString = DW_LiveEdit_UniqueID + '="';
var DW_LiveEdit_IDManagerInstance = null;
var DW_LiveEdit_LiveSource = "";
var DW_LiveEdit_CurrentSimpleDOM = null;
var DW_LiveEdit_ReplacedDOM = null;
var DW_LiveEdit_Failed_Offsets = { "start": -1, "end" : -1};
var DW_LiveEdit_LastSucessString = null;
var DW_LiveEdit_PartialRefreshIsSuspended = false;

function DW_LiveEdit_DummyEditToRetryFailedOffsets() {
    'use strict';
	if (DW_LiveEdit_Failed_Offsets.start !== -1 && DW_LiveEdit_Failed_Offsets.end !== -1 && DW_LiveEdit_LastSucessString) {
		window.DWApplyDOMEditsHasInstruction(); // safer to just call a full refresh if we have failed offsets
	}
}

function DW_LiveEdit_IDManager() {
    'use strict';
    this.DW_LiveEdit_CurrentID = 0;
    this.DW_LiveEdit_IDMap = {};
    this.DW_LiveEdit_IDMap_temp = {};
    this.DW_LiveEdit_ID2dwIDMap = {};
}

DW_LiveEdit_IDManager.prototype.getID = function (newTag, delta, domString) {
    'use strict';
    var ret = -1;
    delta = delta || 0;
	var startNum = parseInt(newTag.start, 10);
    var intKey = startNum + delta;

    if (this.DW_LiveEdit_IDMap[intKey] !== undefined && this.DW_LiveEdit_IDMap[intKey].tag === newTag.tag) {
        ret = this.DW_LiveEdit_IDMap[intKey].id;
    } else if (this.DW_LiveEdit_IDMap_temp[intKey] !== undefined && domString && DW_LiveEdit_CurrentSimpleDOM && DW_LiveEdit_CurrentSimpleDOM.nodeMap) {
        var possibleID = this.DW_LiveEdit_IDMap_temp[intKey].id;
		var oldNode = DW_LiveEdit_CurrentSimpleDOM.nodeMap[possibleID];
		if (oldNode) {
			var oldString =  DW_LiveEdit_LastSucessString ? DW_LiveEdit_LastSucessString.slice(oldNode.start, oldNode.end) : DW_LiveEdit_LiveSource.slice(oldNode.start, oldNode.end);
			//If DW_LiveEdit_LastSucessString is there, it means there was some failure edits, we need to pick last successful string for comparison
			var newString = domString.substr(startNum, oldString.length);
			if (oldString === newString) {
				ret = possibleID;
				this.DW_LiveEdit_IDMap[intKey] = this.DW_LiveEdit_IDMap_temp[intKey];
			}
		}
    }
    return ret;
};

DW_LiveEdit_IDManager.prototype.updateID = function (startPivot, endPivot, diff) {
    'use strict';
    this.DW_LiveEdit_IDMap_temp = {};
    var tempMap = {};
	var key = null;
    for (key in this.DW_LiveEdit_IDMap) {
		if (this.DW_LiveEdit_IDMap.hasOwnProperty(key)) {
			var intKey = parseInt(key, 10);
			if (intKey < startPivot) {
				tempMap[intKey] = this.DW_LiveEdit_IDMap[intKey];
			} else if (intKey >= endPivot) {
				tempMap[intKey + diff] = this.DW_LiveEdit_IDMap[intKey];
			} else {
				this.DW_LiveEdit_IDMap_temp[intKey] = this.DW_LiveEdit_IDMap[intKey];
			}
		}
    }
    this.DW_LiveEdit_IDMap = tempMap;
};

DW_LiveEdit_IDManager.prototype.clearTemp = function () {
    'use strict';
    this.DW_LiveEdit_IDMap_temp = {};
};

DW_LiveEdit_IDManager.prototype.setID = function (newTag, delta) {
    'use strict';
    var ret = -1;
    delta = delta || 0;
    var intKey = parseInt(newTag.start, 10) + delta;

    if (intKey !== undefined && newTag.tag !== undefined) {
        this.DW_LiveEdit_IDMap[intKey] = {'id' : ++this.DW_LiveEdit_CurrentID, 'tag' : newTag.tag};
        ret = this.DW_LiveEdit_CurrentID;
    }
    return ret;
};

var DW_PR_Utils = {

	//Utility function to find the DW_LiveEdit_ID of a tag given the start offset of the tag
	//it searches of the immediate liveedit-id attribute and extracts the attribute value
	find_DWID: function (domString, startOffset) {
		'use strict';
		var index = domString.indexOf('>', startOffset);
		var subString = domString.slice(startOffset, index);
		var idIndex = subString.indexOf(DW_LiveEdit_IDString);
		if (idIndex !== -1) {
			var start = idIndex + DW_LiveEdit_IDString.length;
			var end = subString.indexOf('"', start);
			return subString.slice(start, end);
		}
		return null;
		// Not an error case anymore, while doing code view sync, the tags will not contain dwID	
	},

	findID : function (newTag, domString, offset, shouldUpdateMap) {
		'use strict';
		var dwID = DW_PR_Utils.find_DWID(domString, newTag.start);
		var id = DW_LiveEdit_IDManagerInstance.getID(newTag, offset, domString);
		if (id === -1) {
			id = DW_LiveEdit_IDManagerInstance.setID(newTag, offset);
		}

		if (dwID === null) {
			dwID = "CUSTOM" + id;
			if (!newTag.attributes) {
				newTag.attributes = {};
			}
			newTag.attributes[DW_LiveEdit_UniqueID] = dwID;
		}
		//Incase the tag does not contain any dwID, we need to put it as it is required for identifying element in DOM
		if (shouldUpdateMap) {
			DW_LiveEdit_IDManagerInstance.DW_LiveEdit_ID2dwIDMap[id] = dwID;
		}
		return id;
	},

    //find the tag closest to startOffset and endOffset 
	getParentForOffset: function (startOffset, endOffset) {
		'use strict';
		var nodeMap = DW_LiveEdit_CurrentSimpleDOM.nodeMap;
		var retVal = null;
		var refString =  DW_LiveEdit_LastSucessString || DW_LiveEdit_LiveSource;
		var idString = "";
		var dwIDIndex = refString.lastIndexOf(DW_LiveEdit_IDString, startOffset);
		var tagIndex = dwIDIndex !== -1 ? refString.lastIndexOf('<', dwIDIndex) : -1;
		while (tagIndex !== -1) {
			if (DW_LiveEdit_IDManagerInstance.DW_LiveEdit_IDMap[tagIndex]) {
				idString = DW_LiveEdit_IDManagerInstance.DW_LiveEdit_IDMap[tagIndex].id;
				break;
			} else {
				dwIDIndex = refString.lastIndexOf(DW_LiveEdit_IDString, tagIndex);
				tagIndex = dwIDIndex !== -1 ? refString.lastIndexOf('<', dwIDIndex) : -1;
			}
		}

		if (idString !== "") {
			var parentNode = nodeMap[idString];
			while (parentNode) {
				var start = parentNode.start + parentNode.tag.length;
				var end = parentNode.end - (parentNode.tag.length + 3); //considering the entire endtag

				if (start < startOffset && end > startOffset && start < endOffset && end > endOffset) {
					retVal = parentNode;
					break;
				} else {
					parentNode = parentNode.parent;
				}
			}
		}

		return retVal;
	},

	getOrphanClosingTagCount : function (str) {
		'use strict';
	   /*
		imagine the edit being addition of '</div></div><div><div>' string, in this case the parent to be reconstructed goes to level higher
		so in this function we will figure out if we are closing parent tags and accordingly go higher in order.
		for this we will write a stack to figure out if we closing tags that we did not open
		
		Eg: src="1.png"><img src="2.png"></span><br><span>asdfasdfasdf</span>
		The regex will create >,<img,>,</span,>,<br,>,<span,>,</span,>
		
		The first for loop will try fixing up '>', if an opening tag is found before '>', we will delete it
		hence after first loop: >,<img,</span,<br,<span,</span
		
		Next loop tries to see if there are complete elements, i.e. both open and end, it removes them
		hence after 2nd loop: >,<img,</span,<br
		
		Third loop sees potential tags that string is closing but not opening, in this case they are > & </span
		
		*/
		var count = 0;
		if (str && str !== "") {
			var re = new RegExp(/<\/?[^\s>]+|>/g);
			var matchArray = str.match(re);

			if (matchArray) {
				var index;
				for (index = 1; index < matchArray.length; ++index) {
					var matchStr = matchArray[index];
					var matchPrev = matchArray[index - 1];
					if (matchStr === ">" && matchPrev.charAt(0) === '<') {
						matchArray.splice(index, 1);
						--index;
					}
				}

				for (index = 1; index < matchArray.length; ++index) {
					if (matchArray[index].indexOf("/") !== -1) {
						var startMatch = matchArray[index].toLowerCase().replace(/\//g, '');
						var revIndex;
						for (revIndex = index - 1; revIndex >= 0; --revIndex) {
							var match = matchArray[revIndex].toLowerCase();
							if (match === startMatch) {
								var diff = index - revIndex + 1;
								matchArray.splice(revIndex, diff);
								index = revIndex  - 1;
								break;
							}
						}
					}
				}

				for (index = 0; index < matchArray.length; ++index) {
					var currentMatch = matchArray[index];
					if (currentMatch.indexOf("/") !== -1 || currentMatch === ">") {
						++count;
					}
				}
			}
		}
		return count;
	},

	findPartialDOM: function (startOffset, endOffset, editString) {
		'use strict';
		var commonParent = DW_PR_Utils.getParentForOffset(startOffset, endOffset);
		var mismatch = DW_PR_Utils.getOrphanClosingTagCount(editString);
        while (commonParent && mismatch > 0) {
            commonParent = commonParent.parent;
            --mismatch;
        }
        return commonParent;
	},

    buildNodeMap : function (root) {
		'use strict';
		var nodeMap = {};

		function walk(node) {
			if (node.tagID) {
				nodeMap[node.tagID] = node;
			}
			if (node.isElement()) {
				node.children.forEach(walk);
			}
		}

		walk(root);
		root.nodeMap = nodeMap;
	},

	handleDeletions : function (nodeMap, oldSubtreeMap, newSubtreeMap) {
		'use strict';
		var deletedIDs = [];
		Object.keys(oldSubtreeMap).forEach(function (key) {
			if (!newSubtreeMap.hasOwnProperty(key)) {
				deletedIDs.push(key);
				delete nodeMap[key];
			}
		});
	},

    //it adds the delta to start and end offsets if the offset is greater than the pivot
    //keep pivot=0 if you want it to apply for all offsets
    updateOffsetsWithDelta : function (root, delta, pivot) {
		'use strict';
		var nodeMap = root.nodeMap;
		var nodeID = null;
		for (nodeID in nodeMap) {
			if (nodeMap.hasOwnProperty(nodeID)) {
				var node = nodeMap[nodeID];
				if (node.start !== undefined && node.start >= pivot) {
					node.start += delta;
				}
				if (node.end !== undefined && node.end >= pivot) {
					node.end += delta;
				}
			}
		}
	}
};

//called as soon as the doc is loaded, it takes in the entire code source as parameter
//this acts as a base upon which all edits are applied
//here we construct the SimpleDOM wrt to the source and keep it
function DW_LiveEdit_SetupDoc(source) {
    'use strict';
	source = source.replace(/\r/g, '');
    DW_LiveEdit_LiveSource = source;
    DW_LiveEdit_IDManagerInstance = new DW_LiveEdit_IDManager();

    var Builder = window.DW_HTMLSimpleDOM().Builder;
    var builder = new Builder(DW_LiveEdit_LiveSource, 0, null);
    builder.getID = function (newTag) {
		return DW_PR_Utils.findID(newTag, DW_LiveEdit_LiveSource, 0, true);
	};
    var markCache = {};
    DW_LiveEdit_CurrentSimpleDOM = builder.build(false, markCache);

    if (!DW_LiveEdit_CurrentSimpleDOM) {
        //Inform Dreamweaver to keep a note that this document failed to Partial refresh
        var argObj = {};
        console.error('Could not create SImpleDOM structure mostly cause of malformed HTML');
    }
	//window.DWNotifyPartialRefreshComplete(); //this serves dual purpose, in MAC the AJAX call messes up selection sync info when we ask for the HTML doc. 
	//Also when we do a full refresh, we need to resync the source of LiveDoc and recreate the stylesheet. This call we do the same.
}

//Upon an edit, we pass the startOffset and endOffset of the edit wrt to the base string prior to the edit
//also we pass the new string if any. First we construct the new string and its corresponding SimpleDOM
//compare it with the earlier DOM to generate the edits and finally pass it along
function DW_LiveEdit_DoEdit(startOffset, endOffset, editString, shouldOnlyUpdate, fullRefreshIfFailed) {
	'use strict';
	var newString = "";
	if (DW_LiveEdit_LiveSource) {
		newString = DW_LiveEdit_LiveSource.slice(0, startOffset) + editString + DW_LiveEdit_LiveSource.slice(endOffset); //modified document string
	}

    if (!DW_LiveEdit_CurrentSimpleDOM || DW_LiveEdit_LiveSource.length === 0) {
        //Should never reach here, just in case we do let us refresh the browser
        // Notify Dreamweaver to put the files on server before reloading the page.
		if (!shouldOnlyUpdate) {
			if (fullRefreshIfFailed) {
				window.DWApplyDOMEditsHasInstruction();
			} else if (DW_LiveEdit_LiveSource && newString) {
				//if we are typing and we do not have a structure
				//keep tryin if we can get to correct structure
				//once you do do a refresh so that all structures are reset
				var TempBuilder = window.DW_HTMLSimpleDOM().Builder;
				var tempInstance = new TempBuilder(newString, 0, null);
				tempInstance.getID = function (newTag) {
					return DW_PR_Utils.findID(newTag, newString, 0, true);
				};
				var tempDOM = tempInstance.build(true, {});
				if (tempDOM) {
					window.DWApplyDOMEditsHasInstruction();	//refresh call
				}
				DW_LiveEdit_LiveSource = newString;
			}
		}
        return;
    }

    var delta = endOffset - startOffset - editString.length; //length adjustment of edit
	var endBoundary = endOffset - delta; //for the first time that we store fail offsets
	if (DW_LiveEdit_Failed_Offsets.start !== -1 && DW_LiveEdit_Failed_Offsets.end !== -1 && DW_LiveEdit_LastSucessString) {
		//Assume we failed to sync code view due to errors, we need to retry the same code next time we get an edit.
		//Hence we need to expand the offset to include prior failures
		startOffset = startOffset < DW_LiveEdit_Failed_Offsets.start ? startOffset : DW_LiveEdit_Failed_Offsets.start; //everything before startOffset is original string
		endBoundary = (endOffset > DW_LiveEdit_Failed_Offsets.end ? endOffset : DW_LiveEdit_Failed_Offsets.end) - delta; //everything beyond endBoundary is original string
		var endMatchLength = newString.length - endBoundary;
		//following are the adjusted values as per the original success string
		endOffset = DW_LiveEdit_LastSucessString.length - endMatchLength;
		editString = newString.slice(startOffset, endBoundary);
		delta = endOffset - startOffset - editString.length;
	}

	if (DW_LiveEdit_PartialRefreshIsSuspended) {
		if (DW_LiveEdit_Failed_Offsets.start === -1 && DW_LiveEdit_Failed_Offsets.end === -1) {
			DW_LiveEdit_LastSucessString = DW_LiveEdit_LiveSource; //we will store it on first failure only
		}
        DW_LiveEdit_LiveSource = newString; // we are not updating, but we need to save the changes
        DW_LiveEdit_ReplacedDOM = null;
		DW_LiveEdit_Failed_Offsets = { "start": startOffset, "end" : endBoundary};
		return;
	}

    DW_LiveEdit_IDManagerInstance.updateID(startOffset, endOffset, -delta);

    var prevSubDOM = DW_PR_Utils.findPartialDOM(startOffset, endOffset, editString);
    var diff = null,
        incrementalSuccess = false, /*bool to check if we were successful in doing an incremental update*/
        updateSuccess = false; /*bool to check if we were able to create an updated SimpleDOM structure using the current edit and apply the same*/
    var Builder = window.DW_HTMLSimpleDOM().Builder;
	var updateStructures = null;

    if (prevSubDOM && prevSubDOM.parent) {
        var newSubString = newString.slice(prevSubDOM.start, prevSubDOM.end - delta);
        var parent = prevSubDOM.parent;

        var incBuilder = new Builder(newSubString, 0, null);
        incBuilder.getID = function (newTag) {
			return DW_PR_Utils.findID(newTag, newSubString, prevSubDOM.start, shouldOnlyUpdate);
		};
        var nextSubDOM = incBuilder.build(!fullRefreshIfFailed, {});

        if (parent && nextSubDOM) {
            var childIndex = parent.children.indexOf(prevSubDOM);
            if (childIndex === -1) {
                // This should never happen...
                console.error("couldn't locate old subtree in tree");
            } else {
				//update offsets as per the edit
				DW_PR_Utils.updateOffsetsWithDelta(nextSubDOM, prevSubDOM.start, 0);
				// Build a local nodeMap for the old subtree so the differ can
				// use it.
				DW_PR_Utils.buildNodeMap(prevSubDOM);

				updateStructures = function () {
					// Swap the new subtree in place of the old subtree.
					prevSubDOM.parent = null;
					nextSubDOM.parent = parent;
					parent.children[childIndex] = nextSubDOM;

					//update offsets as per the edit
					DW_PR_Utils.updateOffsetsWithDelta(DW_LiveEdit_CurrentSimpleDOM, -delta, startOffset);

					// Overwrite any node mappings in the parent DOM with the
					// mappings for the new subtree. We keep the nodeMap around
					// on the new subtree so that the differ can use it later.
					$dwJQuery.extend(DW_LiveEdit_CurrentSimpleDOM.nodeMap, nextSubDOM.nodeMap);

					// Clean up the info for any deleted nodes that are no longer in
					// the new tree.
					DW_PR_Utils.handleDeletions(DW_LiveEdit_CurrentSimpleDOM.nodeMap, prevSubDOM.nodeMap, nextSubDOM.nodeMap);

					// Update the signatures for all parents of the new subtree.
					var curParent = parent;
					while (curParent) {
						curParent.update();
						curParent = curParent.parent;
					}
				};

                if (!shouldOnlyUpdate) {
                    diff = window.DW_HTMLDOMDiff().domdiff(prevSubDOM, nextSubDOM);
                }
                incrementalSuccess = true;
                updateSuccess = true;
                DW_LiveEdit_ReplacedDOM = prevSubDOM;
            }
        }
    }

    //incase we couldn't success fully apply an inceremental update, rebuild the DOM and do a complete diff though exoensive
    if (!incrementalSuccess) {

        var domStart = DW_LiveEdit_CurrentSimpleDOM.start + DW_LiveEdit_CurrentSimpleDOM.tag.length;  //considering start of tag including name
        var domEnd = DW_LiveEdit_CurrentSimpleDOM.end - (DW_LiveEdit_CurrentSimpleDOM.tag.length + 3); // considering entire end tags, 3 represents the characters "</>"
        if (startOffset < domStart || endOffset > domEnd) {
            //This happens when the source is broken and we are having a tree that does not contain things fully
            //In this case we can find edits of children of the main tree only
            // Notify Dreamweaver to put the files on server before reloading the page.
			if (fullRefreshIfFailed) {
				if (shouldOnlyUpdate) {
					DW_LiveEdit_CurrentSimpleDOM = null; //Partial Refresh has failed, but ELV operation has happened fine
					//however before next partial refresh we want a full refresh
				} else {
                    // Notify Dreamweaver to put the files on server before reloading the page.
                    window.DWApplyDOMEditsHasInstruction();
                }
            }
            return;
        }

        var fullBuilder = new Builder(newString, 0, null);
        fullBuilder.getID = function (newTag) {
			return DW_PR_Utils.findID(newTag, newString, 0, shouldOnlyUpdate);
		};
        var newDOM = fullBuilder.build(!fullRefreshIfFailed, {});

        if (newDOM) {
            if (!shouldOnlyUpdate) {
                diff = window.DW_HTMLDOMDiff().domdiff(DW_LiveEdit_CurrentSimpleDOM, newDOM);
            }
            DW_LiveEdit_ReplacedDOM = DW_LiveEdit_CurrentSimpleDOM;
			updateStructures = function () {
				DW_LiveEdit_CurrentSimpleDOM = newDOM;
			};
			updateSuccess = true;
        }
    }

    if (diff) {
        //We successfully generated edits implies update has been successful so far
        updateSuccess = window.DW_RemoteFunctions().applyDOMEdits(diff); //we will get a false we were unable to find the element upon which the edit had to be done
    }

    if (updateSuccess) {
		if (updateStructures) {
			updateStructures();
		}

        DW_LiveEdit_IDManagerInstance.clearTemp();
        DW_LiveEdit_LiveSource = newString; // we were able to update our structures hence it is safe to update the source string
        DW_LiveEdit_ReplacedDOM = null;
		DW_LiveEdit_Failed_Offsets = { "start": -1, "end" : -1};
		DW_LiveEdit_LastSucessString = null;
    } else if (fullRefreshIfFailed) {
        if (window.DW_RemoteFunctions().applyDOMEditsHasInstruction()) {
            window.DW_RemoteFunctions().resetApplyDOMEditsHasInstruction();
        }
		if (shouldOnlyUpdate) {
			DW_LiveEdit_CurrentSimpleDOM = null; //Partial Refresh has failed, but ELV operation has happened fine
			//however before next partial refresh we want a full refresh
		} else {
            // Notify Dreamweaver to put the files on server before reloading the page.
            window.DWApplyDOMEditsHasInstruction();
        }
    } else {
		if (DW_LiveEdit_Failed_Offsets.start === -1 && DW_LiveEdit_Failed_Offsets.end === -1) {
			DW_LiveEdit_LastSucessString = DW_LiveEdit_LiveSource; //we will store it on first failure only
		}
		DW_LiveEdit_IDManagerInstance.clearTemp();
        DW_LiveEdit_LiveSource = newString; // we failed to update, but we need to save the changes
        DW_LiveEdit_ReplacedDOM = null;
		DW_LiveEdit_Failed_Offsets = { "start": startOffset, "end" : endBoundary};
	}
}

function DW_LiveEdit_SuspendPartialRefresh() {
    'use strict';
	DW_LiveEdit_PartialRefreshIsSuspended = true;
	//word of caution for using this function, it is caller's responsibilty to resume!
}

function DW_LiveEdit_ResumePartialRefreshIfSuspended() {
    'use strict';
	DW_LiveEdit_PartialRefreshIsSuspended = false;
	if (DW_LiveEdit_Failed_Offsets.start !== -1 && DW_LiveEdit_Failed_Offsets.end !== -1 && DW_LiveEdit_LastSucessString) {
		window.DW_LiveEdit_DoEdit(DW_LiveEdit_Failed_Offsets.start, DW_LiveEdit_Failed_Offsets.start, "", false, true); //Dummy call, we apply edits received in middle
	}
	//On resume we will try committing the changes and if we fail, we will trigger a full refresh, don't use this if that is not what you want.
}

var DW_RefreshCSS = {
	//Recursive logic that finds the appropriate stysheet based on the path
	findStyleSheetImpl : function (styleObject, path) {
		'use strict';
		if (styleObject.href && path) {
            var stylePath = styleObject.href.replace(/:/g, '|').replace(/\|/g, '%7C');
            var rulePath = path.replace(/:/g, '|').replace(/\|/g, '%7C');
            if (rulePath.indexOf(stylePath) !== -1) {
                return styleObject;
            }
		}

		if (!styleObject.cssRules) {
			return null;
		}

		var index = 0;
		for (index = 0; index < styleObject.cssRules.length; ++index) {
			var retVal = null, rule = styleObject.cssRules[index];
			if (rule.type === CSSRule.IMPORT_RULE) {
				retVal = DW_RefreshCSS.findStyleSheetImpl(rule.styleSheet, path);
			}
			if (retVal) {
				return retVal;
			}
		}
	},
	//wrapper for the above recursive function
	findStyleSheet : function (path) {
		'use strict';
		var currentStyleSheet = null, index = 0;
		for (index = 0; index <  document.styleSheets.length; ++index) {
			currentStyleSheet = DW_RefreshCSS.findStyleSheetImpl(document.styleSheets[index], path);
			if (currentStyleSheet) {
				break;
			}
		}
		return currentStyleSheet;
	},
	//CEF is nuts! when we apply a media query and ask it back for the string, it jumbles it up. Hence we also give it our version and get it back
	sanitizeMediaString : function (mQuery) {
		'use strict';
		var ret = "";
		var cssNode = document.createElement('style');
		cssNode.type = 'text/css';
		cssNode.rel = 'stylesheet';
		document.head.appendChild(cssNode);
		var styleObj = cssNode.sheet;
		if (styleObj && styleObj.insertRule && styleObj.deleteRule && styleObj.cssRules) {
			var ruleText = "@media " + mQuery + " {}";
			styleObj.insertRule(ruleText, 0);
			ret = styleObj.cssRules[0].media.mediaText;
			styleObj.deleteRule(0);
		}
		document.head.removeChild(cssNode);
		return ret;
	},

	compareFullRule : function (string1, string2) {
		'use strict';
		var ret = false;
		if (string1 && string2) {
			string1 = string1.toLowerCase();
			string2 = string2.toLowerCase();

			//incase we have url, we would want to compare only the file names, as rest might be mangled
			var urlRegex = /url.*[\\\/]/g;
			string1 = string1.replace(urlRegex, "url(");
			string2 = string2.replace(urlRegex, "url(");

			//Now for the rest we ignore space,",',;,rgb and hex color values
			var regexPattern =  /\s|;|"|'|rgb\(.*\)|(#[0-9a-f]*)(?![\s\S]*\{)/g;
			ret = string1.replace(regexPattern, "") === string2.replace(regexPattern, "");
		}
		return ret;
	},

	compareRuleStrings : function (string1, string2) {
		'use strict';
		var ret = false;
		if (string1 && string2) {
			//clear out the spaces
			string1 = string1.replace(/\s/g, '');
			string2 = string2.replace(/\s/g, '');

			if (string1.length > 0 && string2.length > 0) {
				if (string1.charAt(0) === '@' && string2.charAt(0) === '@') {
					//could be @import, @fontface, @charset, need to compare fully
					ret = DW_RefreshCSS.compareFullRule(string1, string2);
				} else {
					//it is a normal style rule, kick out everything in between the curly braces and compare only selectorText
					var regexPattern = /\{[\s\S]*\}/g;
					ret = string1.replace(regexPattern, '') === string2.replace(regexPattern, '');
				}
			}
		}
		return ret;
	},
	//if our selector is in a media rule, we try to find the media rule first
	findMediaRule : function (mQuery, styleObj) {
		'use strict';
		var ret = null;
		if (mQuery && mQuery.length > 0 && styleObj && styleObj.cssRules) {
			var index = 0;
			var rules = styleObj.cssRules;
			var modQuery = DW_RefreshCSS.sanitizeMediaString(mQuery);
			for (index = 0; index < rules.length; ++index) {
				if (rules[index].type === CSSRule.MEDIA_RULE && rules[index].media && rules[index].media.mediaText === modQuery) {
					ret = rules[index];
					break;
				}
			}
		}
		return ret;
	},
	//the important function, given a css Selector, we add the style.
	insertRule : function (mediaText, ruleText, path) {
		'use strict';
		var sheet = DW_RefreshCSS.findStyleSheet(path);
		if (sheet) {
			var parent = null;
			if (mediaText && mediaText.length > 0) {
				parent = DW_RefreshCSS.findMediaRule(mediaText, sheet);
				if (!parent) {
					parent = sheet;
					ruleText = '@media ' + mediaText + ' { ' + ruleText + ' }'; //if the selector is in a media query we should do so!
				}
			} else {
				parent = sheet;
			}

			if (parent && parent.cssRules) {
				var insertIndex = ruleText.indexOf("@import") !== -1 ? 0 : parent.cssRules.length;//if we are adding an import statement, add it at start, otherwise end
				parent.insertRule(ruleText, insertIndex);
                
                if (navigator && navigator.userAgent &&
                        navigator.userAgent.toLowerCase().indexOf('safari') !== -1 &&
                        navigator.userAgent.toLowerCase().indexOf('chrome') === -1) {
                    //this is a hack for safari where, sometimes adding the style doesn't reflect right away, jus flip the disabled attribute to get this working
                    parent.disabled =  !parent.disabled;
                    parent.disabled =  !parent.disabled;
                }
			}
		}
	},
	//the important function, given a css Selector, we delete the style.
	deleteRule : function (mediaText, ruleText, path) {
		'use strict';
		var sheet = DW_RefreshCSS.findStyleSheet(path);
		if (sheet) {
			var parent = null;
			if (mediaText && mediaText.length > 0) {
				parent = DW_RefreshCSS.findMediaRule(mediaText, sheet);
			} else {
				parent = sheet;
			}

			if (parent && parent.cssRules) {
				var matches = []; //we will store all possible matches for the rule, if there are more than one we will decide which to delete later.
				var index = 0;
				var rules = parent.cssRules;
				for (index = 0; index < rules.length; ++index) {
					if (rules[index].type !== CSSRule.MEDIA_RULE && DW_RefreshCSS.compareRuleStrings(rules[index].cssText, ruleText)) {
						matches.push(index);
					}
				}
				if (matches.length === 1) {
					parent.deleteRule(matches[0]);
				} else if (matches.length > 1) {
					for (index = 0; index < matches.length; ++index) {
						if (DW_RefreshCSS.compareFullRule(rules[matches[index]].cssText, ruleText)) {
							parent.deleteRule(matches[index]);
							break;
						}
					}
				}
			}
		}
	}
};

function DWApplyDOMEditsHasInstruction() {
    'use strict';
    window.location.reload(true);
}