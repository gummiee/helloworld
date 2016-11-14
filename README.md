# helloworld

<h2>My name is Linh</h2>
<h3>How does this work?</h3>

<script type="text/javascript">

(function( $, document, wb ) {
	"use strict";

		
	var componentName = "wb-fieldflow",
		selector = "." + componentName,
		formComponent = componentName + "-form",
		subComponentName = componentName + "-sub",
		crtlSelectClass = componentName + "-init",
		crtlSelectSelector = "." + crtlSelectClass,
		subSelector = "." + subComponentName,
		basenameInput = componentName + wb.getId(),
		labelClass = componentName + "-label",
		headerClass = componentName + "-header",
		selectorForm = "." + formComponent,
		initEvent = "wb-init" + selector,
		drawEvent = "draw" + selector,
		actionEvent = "action" + selector,
		submitEvent = "submit" + selector,
		submitedEvent = "submited" + selector,
		readyEvent = "ready" + selector,
		cleanEvent = "clean" + selector,
		createCtrlEvent = "createctrl" + selector,
		cleanJQData = componentName + "-clean",
		registerJQData = componentName + "-register", // Data that contain all the component registered (to the form element and component), used for executing action upon submit
		configData = componentName + "-config",
		triggerJQData = componentName + "-trigger",
		pushJQData =  componentName + "-push",
		submitJQData =  componentName + "-submit", // List of action to perform upon form submission
		actionData =  componentName + "-action", // temp for code transition
		bindtoData = componentName + "-bindto", // To carry the id of the element that it is binded to
		originData =  componentName + "-origin", // To carry the plugin origin ID, any implementation of "createCtrlEvent" must set that option.
		flagOptValueData =  componentName + "-flagoptvalue",
		
		$document = wb.doc,
		
		defaults = {
			toggle: {
				stateOn: "visible", // For toggle plugin
				stateOff: "hidden"  // For toggle plugin
			},
			// noForm //if true jj-down wont add a form wrapper
			// noDefaultAjax // if true jj-down will assume the "jj-down" value are needed for form submission and server side manipulation (Please not that when used, the name on the select must be set and that information must be supplied through config)
			
			// unhideelm // [JQuery Selector] If specified, the class "hidden" will be removed on init on that element
			// hideelm // [JQuery Selector] If specified, the class "hidden" will be added on init on that element
			i18n: 
			{
				"en": {
					btn: "Continue", // Action button
					emptysel: "Make your selection...", // text use for the first empty select
					required: "required"// text for the required label
				},
				"fr": {
					btn: "Allez",
					emptysel: "Sélectionnez dans la liste...", // text use for the first empty select
					required: "obligatoire" // text for the required label
				}
			}
		},
		//i18n,
		
		/**
		* @method init
		* @param {jQuery Event} event Event that triggered the function call
		*/
		init = function( event ) {
			// Start initialization
			// returns DOM object = proceed with init
			// returns undefined = do not proceed with init (e.g., already initialized)
			var elm = wb.init( event, componentName, selector ),
				$elm, elmId,
				i18nCache,
				wbDataElm,
				config,
				i18n;

			if ( elm ) {
				$elm = $(elm);
				
				elmId = elm.id;
				
				// Set default i18n information
				
				if ( defaults.i18n[ wb.lang ] ) {
					defaults.i18n = defaults.i18n[ wb.lang ];
				}

				// Extend this data with the contextual default
				wbDataElm = wb.getData( $elm, componentName );
				if (wbDataElm && wbDataElm.i18n ){
					wbDataElm.i18n = $.extend( {}, defaults.i18n, wbDataElm.i18n );
				}
				config = $.extend( {}, defaults, wbDataElm );

				// Set the data to the component, if other event need to have access to it.
				$elm.data( configData, config );
				
				i18n = config.i18n;
				
				// Todo: move this shim to the wb.core
				// Add startWith function (ref: https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/String/startsWith)
				if ( !String.prototype.startsWith ) {
					String.prototype.startsWith = function ( searchString, position ) {
						position = position || 0;
						return this.substr( position, searchString.length ) === searchString;
					};
				}
				
				
				// Transform the list into a select, use the first paragrap content for the label, and extract for i18n the name of the button action.
				var // formID = wb.getId(),
					bodyID = wb.getId(),
					stdOut,
					formElm, $form,
					dataDraw = {};
				
				if ( config.noForm ) {
					stdOut = "<div class='mrgn-tp-md'><div id='" + bodyID + "'></div></div>";
					// stdOut = stdOut + '<div id="' + formID + 'out" class="row mrgn-tp-md mrgn-bttm-md"></div>';
					
					// Need to add the class="formComponent" to the div that wrap the form element.
					formElm = elm.parentElement;
					while ( formElm.nodeName !== "FORM" ) {
						formElm = formElm.parentElement;
					}

					$(formElm.parentElement).addClass( formComponent );
					
					
				} else {
					stdOut = "<div class='wb-frmvld " + formComponent + "'><form><div id='" + bodyID + "'>";
					stdOut = stdOut + '</div><input type="submit" value="' + i18n.btn + '" class="btn btn-primary" /> </form></div>';
					// stdOut = stdOut + '<div id="' + formID + 'out" class="row mrgn-tp-md mrgn-bttm-md"></div>';
				}
				$elm.addClass("hidden");
				stdOut = $(stdOut);
				$elm.after(stdOut);
				
				
				if ( ! config.noForm ) {
					
					formElm = stdOut.find("form");
					
					//$(".wb-frmvld." + formComponent).trigger( "wb-init.wb-frmvld" );
					stdOut.trigger( "wb-init.wb-frmvld" );
				}
				
				$form = $(formElm);
				
				
				// Register this plugin within the form, this is to manage form submission
				pushData( $form, registerJQData, elmId );
				
				
				if ( !config.outputctnrid ) { // Output container ID
					config.outputctnrid = bodyID;
				}
				
				if ( !config.source ) {
					config.source = elm; // We assume th container have the source
				}
				
				if ( !config.srctype ) {
					config.srctype = componentName;
				}
				
				// Trigger the drop down loading
				$elm.trigger( config.srctype + "." + drawEvent, config );
				
				// Do requested DOM manipulation 
				if ( config.unhideelm ) {
					$( config.unhideelm ).removeClass( "hidden" );
				}
				if ( config.hideelm ) {
					$( config.hideelm ).addClass( "hidden" );
				}
				
				// Identify that initialization has completed
				wb.ready( $elm, componentName );
				
				if ( config.action ) {
					config.form = $form.get(0);
					$elm.trigger( config.action + "." + readyEvent, config );
				}
			}
		},
		
		pushData = function($elm, prop, data, reset){
			var dtCache = $elm.data( prop );
			if ( !dtCache || reset ) {
				dtCache = [];
			}
			dtCache.push( data );
			return $elm.data( prop, dtCache );
		},
		
		toRadio = function( $label, items ){
			var htmlOut,
				i, i_len,
				curr_itm, itmID,
				i18nRequired = "required",
				nameRadioSet = basenameInput + wb.getId();
			
			// Open Element
			htmlOut = "<fieldset><legend class='required'><span class='field-name'>" + $label.html() + "</span> <strong>(" + i18nRequired + ")</strong></legend>";

			// Body
			i_len = items.length;
			for ( i = 0; i !== i_len; i += 1 ) {
				curr_itm = items[ i ];
				itmID = wb.getId();
				
				htmlOut = htmlOut + "<div class='radio'>" +
					"<label for='" + itmID + "'><input type='radio' name='" + nameRadioSet + "' value='" + curr_itm.value + "' id='" + itmID + "' required />" +
					curr_itm.label + "</label></div>"
			}
			
			
			// Closing
			htmlOut = htmlOut + '</fieldset>';
			return htmlOut;
			
		},/*,
		toSelectMulti = function(){},
		toCheckbox = function(){},
		*/
		
		cleaning = function( $orgin, $elm ){
			var i, i_len, dtCache, i_cache;
			dtCache = $elm.data( cleanJQData );
			if ( dtCache ) {
				i_len = dtCache.length;
				for( i = 0; i !== i_len; i += 1 ) {
					i_cache = dtCache[ i ];
					$orgin.trigger(i_cache.action + "." + cleanEvent, i_cache );
				}
			}
		};
		
	$document.on( "toggle." + cleanEvent, selector, function( event, data ) {
		var $cacheOptSel = data.$elm;
		
		// Doing an add and remove "wb-toggle" class in order to avoid the click event added by toggle plugin
		$cacheOptSel.addClass("wb-toggle");
		$cacheOptSel.trigger( "toggle.wb-toggle", data.toggle ); 
		$cacheOptSel.removeClass("wb-toggle");
	});
	
	$document.on( "ajax." + cleanEvent, selector, function( event, data ) {
		console.log(data);
		$( data.selector ).empty();
	});
	
	$document.on( "tblfilter." + cleanEvent, selector, function( event, data ) {
		$("#" +  data.bind ).dataTable({ "retrieve": true }).api().column( data.column ).search('').draw();
	});
	
	// Load content after the user have choosen an option
	$document.on( "change", selectorForm + " " + crtlSelectSelector, function( event ) {	

		var elm = event.currentTarget,
			$elm = $(elm),
			data,
			dataJQ,
			selCurrentElm, cacheAction,
			i, i_len, j, j_len, dtCached, dtCachedAction, dtCachedTarget,
			itmToClean = $elm.nextAll(),
			$orgin = $("#" + elm.getAttribute( "data-" + originData ) );
			

		
		// The parent element of "elm" should be the form body container.

		// Check if a new sub-list need to be created, otherwise remove all subsequent select
		
		var $optSel = $elm.find( ':checked', $elm), 
			optSel = $optSel,
			optValue = optSel.value,
			componentName_len = componentName.length,
			form = $elm.get(0).form;
		

		//
		// 1. Cleaning
		//
		i_len = itmToClean.length;
		for ( i = i_len; i !== 0; i -= 1 ){
			cleaning( $orgin, $( itmToClean[ i ] ) );
		}
		$elm.nextAll().remove();
		cleaning( $orgin, $elm );
		
		// Remove any action that is pending for form submission
		$elm.data( submitJQData, [] );

		
		
		//
		// 2. Get defined actions
		//

		var actions = [],
			nowActions = [],
			postActions = [], postAction_len,
			bindTo,
			bindToElm;
		
		
		// From the component
		actions = $elm.data( pushJQData );
		if ( ! actions ){
			actions = [];
		}

		// For each the binded elements that are selected
		for( i = 0, i_len = $optSel.length; i !== i_len; i += 1 ){
			selCurrentElm = $optSel.get( i );
			bindTo = selCurrentElm.getAttribute("data-" + bindtoData);
			if ( bindTo ) {
				// Retreive action set on the binded element
				bindToElm = document.getElementById( bindTo );
				cacheAction = parseActionStr( bindToElm.getAttribute( "data-" + componentName ) );
				cacheAction = addCommonProperty( cacheAction, "selElm", selCurrentElm );
				actions = actions.concat( cacheAction );
			}

			// From the selected options
			cacheAction = parseActionStr( selCurrentElm.getAttribute( "data-" + actionData ) );
			cacheAction = addCommonProperty( cacheAction, "selElm", selCurrentElm );
			actions = actions.concat( cacheAction );
		}
		
		// If there is no action, do nothing
		if ( ! actions.length ) {
			return true;
		}
		
		
		//
		// 3. Sort action 
		// 			array1 = Action to be executed now
		//			array2 = Action to be postponed for later use
		for( i = 0, i_len = actions.length; i !== i_len; i += 1 ) {
			dtCached = actions[ i ];
			dtCachedTarget = dtCached.target;
			if ( !dtCachedTarget || dtCachedTarget === bindTo ) {
				nowActions.push( dtCached );
			} else {
				postActions.push( dtCached );
			}
		}
		
		//
		// 4. Execute action for the current item
		//
		postAction_len = postActions.length;
		for( i = 0, i_len = nowActions.length; i !== i_len; i += 1 ) {
			dtCached = nowActions[ i ];
			dtCached.origin = bindToElm;
			dtCached.provEvt = elm;
			dtCached.form = form;
			if ( postAction_len ){
				dtCached.actions = postActions;
			}
			$orgin.trigger( dtCached.action + "." + actionEvent, dtCached );
		}
		
		return true;
	});
	
	function addCommonProperty( arr, propName, propValue ) {
		var i, i_len;
		
		for( i = 0, i_len = arr.length; i !== i_len; i += 1 ) {
			arr[ i ][ propName ] = propValue;
		}
		return arr;
	};
	
	
	/* 
	 * handlebars action
	 * 
	 * Add a parameter to the detination URL like "?name=value"
	 */
	 $document.on( "handlebars." + actionEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		actionEvent = {
			actions: "",
			deferactions: [], // Array of action that will be executed later on
			target: "", // If not set, target is current provEvt
			provEvt: DOM // DOM element of the controler that initiated the change event
			origin: DOM || Undefined // DOM element of where the action is binded to
			selElm: DOM // DOM element that represent the selection (usually this will be an form/input field)
		}*/
		/*
		 * Data for handlebasrEvent interface
		 *
		handlebasrEvent = {
			// see bellow
		}
		call example:
		{"action": "handlebars", ...}
		*/
		
		throw "Not implemented";

		// Template out - http://handlebarsjs.com
		
		// Load JSON DATA ????
		// preload Handlebars library?....
		
		
		// Contextual data
		var dtHandlebars = {
			precompiled: false, // If true we will use the run time
			container: "jQuery Selector", // Where the template will be added
			type: "replace", // Type of insertion
			url: "/url/of/the/template", // url of the template
			data: { }, // pre-defined data that will be use on the template (It will be extended [Priority: plugin level, item level, query level, item with overwrite instruction)
			overwrite: false, // Data will overwrite whatever before
			useQuery: true, // query action will impact on parsing the result in the template
			trigger: false // Run WET feature on the result content
		};
		
		
		// Fetch the template (if not pre-fetched)
		// Load Handlebars.js (if not pre-loaded) 
		// Compile the template (if it not pre-compiled)
		// Render
		// Add content in the container by following the type rule.
	});
	
	
	/* 
	 * Redirection action
	 * 
	 * Add a parameter to the detination URL like "?name=value"
	 */
	$document.on( "redir." + actionEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		submitEvent = {
			actions: "",
			provEvt: DOM // DOM element of the controler that have set the action
			origin: DOM // wb-fieldflow component
			form: // DOM form element on which the submit is appening.
		}*/
		/*
		 * Data for redirEvent interface
		 *
		redirEvent = {
			url: '', // URL to redirect to
		}
		call example:
		{"action": "redir", "url":"http://an/location"}
		*/
		// Set "data.preventSubmit = true" in order to prevent form submission
		// Set the REDIR action on the component itself, only one action "redir" could be set, so we trash any other action that may have been set before
		pushData( $(data.provEvt), submitJQData, data, true );
	});
	
	/* 
	 * Redirection action
	 * 
	 * Add a parameter to the detination URL like "?name=value"
	 */
	$document.on( "redir." + submitEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		submitEvent = {
			actions: "",
			provEvt: DOM // DOM element of the controler that have set the action
			origin: DOM // wb-fieldflow component
			form: // DOM form element on which the submit is appening.
		}*/
		/*
		 * Data for redirEvent interface
		 *
		redirEvent = {
			url: '', // URL to redirect to
		}
		call example:
		{"action": "redir", "url":"http://an/location"}
		*/
		var form = data.form,
			url = data.url;

		if ( url ) {
			form.setAttribute( "action", url );
		}
	});
	
	
	/* 
	 * Querying action
	 * 
	 * Add a parameter to the detination URL like "?name=value"
	 */
	 $document.on( "query." + actionEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		actionEvent = {
			actions: "",
			deferactions: [], // Array of action that will be executed later on
			target: "", // If not set, target is current provEvt
			provEvt: DOM // DOM element of the controler that initiated the change event
			origin: DOM || Undefined // DOM element of where the action is binded to
			selElm: DOM // DOM element that represent the selection (usually this will be an form/input field)
		}*/
		/*
		 * Data for queryEvent interface
		 *
		queryEvent = {
			name: '', // The name of the parameter
			value: '', // The value to set for the parameter
		}
		{"action": "query", "name":"lob", "value":"eepr"}
		*/
		
		var selectElm = data.selElm,
			fieldName = data.name,
			fieldValue = data.value;
		
		// Add a type property, value are JSON,...
		
		if ( fieldName ) {
			data.provEvt.setAttribute( "name", fieldName);
		}
		
		if ( fieldValue ) {
			selectElm.value = fieldValue;
		}
		
		// Add a flag to know the option value was inserted
		selectElm.setAttribute( "data-" + flagOptValueData, flagOptValueData );
	});

	/* Perfoming AJAX action
	 * 
	 * @elm : Element that triggered this action
	 * @dt: Data structure defining this ajax loading configuration
	 * 
	 */
	 $document.on( "ajax." + actionEvent, selector, function( event, data ) {
	//function onAjax (elm, dt){
		/*
		 * Data for actionEvent interface
		 *
		actionEvent = {
			actions: "",
			deferactions: [], // Array of action that will be executed later on
			target: "", // If not set, target is current provEvt
			provEvt: DOM // DOM element of the controler that initiated the change event
			origin: DOM || Undefined // DOM element of where the action is binded to
			selElm: DOM // DOM element that represent the selection (usually this will be an form/input field)
		}*/
		/*
		 * Data for ajaxEvent interface
		 *
		ajaxEvent = {
			url: '', // URL of the content to be ajaxed
			tupe: '', // Type of content insertion in the page
			container: '', // Container or where the content will be inserted
			trigger: false, // Boolean: Enhance wet component on ajaxed content
			clean: '' // cleaning action upon change
		}
		{action: 'ajax', url: 'ajax/jj-down-result-1.html', 'type':'before', 'container':'#alternate-version2', trigger, clean}
		*/
		
		var provEvt = data.provEvt,
			$container;
		
		if ( !data.live ){
			data.preventSubmit = true;
			pushData( $(provEvt), submitJQData, data );
		} else {
			if ( ! data.container ) {
				// Create the container next to component
				$container = $( "<div></div>" );
				$( provEvt.parentNode ).append( $container );
				data.container = $container.get(0);
			}
			$(event.target).trigger( "ajax." + submitEvent, data );
		}
		
	});

	
	
	/* 
	 * Ajax action
	 * 
	 * Add a parameter to the detination URL like "?name=value"
	 */
	$document.on( "ajax." + submitEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		submitEvent = {
			actions: "",
			provEvt: DOM // DOM element of the controler that have set the action
			origin: DOM // wb-fieldflow component
			form: // DOM form element on which the submit is appening.
		}*/
		
		var provEvt = data.provEvt,
			$container, containerID, ajxType;
		
		if ( ! data.container ) {
			containerID = wb.getId();
			$container = $( "<div id='" + containerID + "'></div>" );
			$( data.form ).append( $container );
			data.clean = "#" + containerID;
		} else {
			$container = $( data.container );
		}
		
		if ( data.clean ) {
			// Stack the cleaning task for execution upong next onChange event.
			pushData( $(provEvt), cleanJQData, {
				action: "ajax",
				selector: data.clean
			});
		}

		if ( data.trigger ) {
			$container.attr( "data-trigger-wet", "true" );
		}
		
		ajxType = data.type ? data.type : "replace";
		$container.attr( "data-ajax-" + ajxType, data.url );

		$container.one( "wb-contentupdated", function( event, data ) {
			var updtElm = event.currentTarget,
				trigger = updtElm.getAttribute( "data-trigger-wet");

			updtElm.removeAttribute( "data-ajax-" + data["ajax-type"] );
			
			if ( trigger ) {
				$( updtElm )
					.find( wb.allSelectors )
						.addClass( "wb-init" )
						.filter( ":not(#" + updtElm.id + " .wb-init .wb-init)" )
							.trigger( "timerpoke.wb" );
				updtElm.removeAttribute( "data-trigger-wet");
			}
		} );
		$container.trigger( "wb-update.wb-data-ajax" );
	});
	
	// Toggle an items
	$document.on( "toggle." + actionEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		actionEvent = {
			actions: "",
			deferactions: [], // Array of action that will be executed later on
			target: "", // If not set, target is current provEvt
			provEvt: DOM // DOM element of the controler that initiated the change event
			origin: DOM || Undefined // DOM element of where the action is binded to
			selElm: DOM // DOM element that represent the selection (usually this will be an form/input field)
		}
		 */
		/*
		 * Data for tblfilterEvent interface
		 *
		tblfilterEvent = {
			bind:		// [required] Id of the data table where the filtering would be applied to
			column: 	// [required] Column number (int) or column selector [see: https://datatables.net/reference/type/column-selector]
						//	(It could be optional if we can specify the select if for filtering one column only.)
			value: 		// [required] Text to search [see: https://datatables.net/reference/api/column%28%29.search%28%29]
			regex:		// [optional] True | False (default). Define if the value is a regular expression or not [see: https://datatables.net/reference/api/column%28%29.search%28%29]
			smart:		// [optional] True (default) | False. If we should do smart search of not [see: https://datatables.net/reference/api/column%28%29.search%28%29]
			caseinsen:	// [optional] True (default) | false. If value are case insensitive [see: https://datatables.net/reference/api/column%28%29.search%28%29]
		}
		 */
		if ( !data.live ){
			data.preventSubmit = true;
			pushData( $(data.provEvt), submitJQData, data );
		} else {
			$(event.target).trigger( "toggle." + submitEvent, data );
		}
	});
	
	
	/* 
	 * Toggle action
	 * 
	 * Add a parameter to the detination URL like "?name=value"
	 */
	$document.on( "toggle." + submitEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		submitEvent = {
			actions: "",
			provEvt: DOM // DOM element of the controler that have set the action
			origin: DOM // wb-fieldflow component
			form: // DOM form element on which the submit is appening.
		}*/
		var $origin = $( data.origin ),
			config = $( event.target ).data( configData ),
			toggleOpts = data.toggle;
		
		// For simple toggle call syntax
		if (toggleOpts && typeof toggleOpts === "string" ) {
			toggleOpts = { selector: toggleOpts };
		}
		toggleOpts = $.extend( {}, toggleOpts, config.toggle );

		// Doing an add and remove "wb-toggle" class in order to avoid the click event added by toggle plugin
		$origin.addClass( "wb-toggle" );
		$origin.trigger( "toggle.wb-toggle", toggleOpts ); 
		//$origin.removeClass( "wb-toggle" );
		
		// Set the cleaning task
		toggleOpts.type = "off";
		pushData( $(data.provEvt), cleanJQData, {
			action: "toggle",
			$elm: $origin,
			toggle: toggleOpts
		} );
	});
		
	
	// Insert a control next to it
	$document.on( "append." + actionEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		actionEvent = {
			actions: "",
			deferactions: [], // Array of action that will be executed later on
			target: "", // If not set, target is current provEvt
			provEvt: DOM // DOM element of the controler that initiated the change event
			origin: DOM || Undefined // DOM element of where the action is binded to
			selElm: DOM // DOM element that represent the selection (usually this will be an form/input field)
		}
		 */
		/*
		 * Data for appendEvent interface
		 *
		appendEvent = {
			selector:		// Selector of which component to be inserted
			srctype: // Type of object to be inserted (tblfilter, wb-fieldflow)
		}
		 */
		if ( event.namespace === actionEvent ) {
			var subId,
				srctype = data.srctype ? data.srctype : componentName,
				$appendComponent;
			
			data.container = data.provEvt.parentNode.id;
			
			// Get the selector ID or add one if not exist
			if ( data.selector ) {
				subId = $( data.selector ).get(0).id;
				if (! subId) {
					subId = wb.getId();
					$( data.selector ).get(0).id = subId;
				}
				$( data.selector ).addClass( subComponentName );

				//srctype = componentName;
				
				$appendComponent = $( data.selector ).addClass( componentName );
				// $appendComponent.data( pushJQData, data.deferactions );
				
				data.source = "#" + subId;
			}
			
			
			//data.actions = arrPostAction;
			
			$( event.currentTarget ).trigger( srctype + "." + drawEvent, data );
		}
	});
	
	
	// Apply the filtering
	$document.on( "tblfilter." + actionEvent, selector, function( event, data ) {
		/*
		 * Data for actionEvent interface
		 *
		actionEvent = {
			actions: "",
			deferactions: [], // Array of action that will be executed later on
			target: "", // If not set, target is current provEvt
			provEvt: DOM // DOM element of the controler that initiated the change event
			origin: DOM || Undefined // DOM element of where the action is binded to
			selElm: DOM // DOM element that represent the selection (usually this will be an form/input field)
		}
		 */
		/*
		 * Data for tblfilterEvent interface
		 *
		tblfilterEvent = {
			bind:		// [required] Id of the data table where the filtering would be applied to
			column: 	// [required] Column number (int) or column selector [see: https://datatables.net/reference/type/column-selector]
						//	(It could be optional if we can specify the select if for filtering one column only.)
			value: 		// [required] Text to search [see: https://datatables.net/reference/api/column%28%29.search%28%29]
			regex:		// [optional] True | False (default). Define if the value is a regular expression or not [see: https://datatables.net/reference/api/column%28%29.search%28%29]
			smart:		// [optional] True (default) | False. If we should do smart search of not [see: https://datatables.net/reference/api/column%28%29.search%28%29]
			caseinsen:	// [optional] True (default) | false. If value are case insensitive [see: https://datatables.net/reference/api/column%28%29.search%28%29]
		}
		 */
		// Note: There is no action on submit because this table filtering mode is only supported in live mode, Technically this is to avoid conflict with datatables API on re-draw, like the use chose filtering options, but do not apply them and they go applied upon table re-order.
		if ( event.namespace === actionEvent ) {
			var bindTo = data.bind,
				$datatable = $("#" + bindTo).dataTable({ "retrieve": true }).api(),
				$provEvt = $(data.provEvt),
				column = data.column,
				colInt = parseInt( column, 10 ),
				regex = !!data.regex,
				smart = ( !data.smart ) ? true : !!data.smart,
				caseinsen = ( !data.caseinsen ) ? true : !!data.caseinsen;

			column = ( colInt === true ) ? colInt : column;
			
			$datatable.column( column ).search( data.value, regex, smart, caseinsen ).draw();

			// Add a clean up task
			pushData( $provEvt, cleanJQData, {
				action: "tblfilter",
				bind: bindTo,
				column: column
			} );
			
		}
	});
	
		
	/* 
	 * Table filtering action
	 * 
	 
	$document.on( "tblfilter." + submitEvent, selector, function( event, data ) {
		$("#" + data.bind).dataTable({ "retrieve": true }).api().draw();
	});
	*/
	
	function parseActionStr( action ){
		var data = [ ];

		
		if ( action ){
			action = $.trim( action );
		} else {
			return data;
		}
		if ( action && ( action.startsWith( "{" ) || action.startsWith( "[" ) ) ) {
			try {
				data = JSON.parse( action );
			} catch ( error ) {
				$.error( "Bad JSON array" );
			}
			if ( ! $.isArray( data ) ) {
				data = [ data ];
			}
		} else {
			data = [ {
				"action": "ajax", // If no action defined, we will assume it means "ajax"
				"url": action
			} ];
		}
		return data;
	}
	
	// Load content after the user have choosen an option
	$document.on( "change", selectorForm + " input[type=radio]", function( event ) {	

	var elm = event.currentTarget,
			$elm = $(elm);

		
		$(elm.form).attr( "action", $elm.val() );
		
	});
	
	
	
	// Load content after the user have choosen an option
	$document.on( "submit", selectorForm + " form", function( event ) {	
		/*
		// Stop propagation of the activate event
		event.preventDefault();
		if ( event.stopPropagation ) {
			event.stopImmediatePropagation();
		} else {
			event.cancelBubble = true;
		}
		*/
			
		var elm = event.currentTarget,
			$elm = $(elm),
			data,
			wbFieldFlowRegistered = $elm.data( registerJQData ),
			i, i_len = wbFieldFlowRegistered ? wbFieldFlowRegistered.length : 0,
			$wbFieldFlow, fieldOrigin,
			lstFieldFlowPostEvent = [],
			componentRegistered, $componentRegistered, $origin, lstOrigin = [],
			settings,
			j, j_len,
			$field,
			m, m_len, m_cache,
			actions,
			retComponentSubmit,
			preventSubmit = false, lastProvEvt;
		
		// Run the cleaning on the current items
		if ( i_len ) {
			$wbFieldFlow = $( "#" + wbFieldFlowRegistered[ i_len - 1 ] );
			fieldOrigin = $wbFieldFlow.data( registerJQData );
			cleaning( $wbFieldFlow, $( "#" + fieldOrigin[ fieldOrigin.length - 1 ] ) );
		}

		// For each wb-fieldflow component, execute submiting task.
		for( i = 0; i !== i_len; i += 1 ){
			$wbFieldFlow = $("#" + wbFieldFlowRegistered[ i ]);
			componentRegistered = $wbFieldFlow.data( registerJQData );
			j_len = componentRegistered.length;
			for( j = 0; j !== j_len; j += 1 ){
				$componentRegistered = $( "#" + componentRegistered[ j ] );
				// Get source
				$origin = $( "#" + $componentRegistered.data(originData) );
				lstOrigin.push( $origin );
				actions = $componentRegistered.data( submitJQData );
				if ( actions ) {
					for ( m = 0, m_len = actions.length; m !== m_len; m += 1 ) {
						m_cache = actions[ m ];
						m_cache.form = elm;
						retComponentSubmit = $wbFieldFlow.trigger( m_cache.action + "." + submitEvent, m_cache );
						lstFieldFlowPostEvent.push( {
							$elm: $wbFieldFlow,
							data: m_cache
						});
						preventSubmit = preventSubmit || m_cache.preventSubmit;
						lastProvEvt = m_cache.provEvt;
					}
				}
			}
		}
		
		// Before to submit, remove jj-down accessesory control
		$elm.find( "[name^=" + basenameInput + "]" ).removeAttr( "name" );

		// Check the form action, if there is query string, do split it and create hidden field for submission
		// The following is may be simply caused by a cross-server security issue generated by the browser itself
		var frmAction, idxQueryDelimiter;
		frmAction = $elm.attr( "action" );
		if ( frmAction ) {
			idxQueryDelimiter = frmAction.indexOf("?");
			if ( idxQueryDelimiter > 0 ) {
				// Split the query string and create hidden input.
				var queryString = frmAction.substring( idxQueryDelimiter + 1 ),
					params = queryString.split('&'),
					i, i_len, cacheParam,
					items;
				
				i_len = params.length;
				for( i = 0; i !== i_len; i += 1 ){
					cacheParam = params[ i ];
					
					if ( cacheParam.indexOf( "=" ) > 0 ) {
						items = cacheParam.split('=', 2);
						
						$elm.append("<input type='hidden' name='" + items[0] + "' value='" + items[1] + "' />");
					} else {
						$elm.append("<input type='hidden' name='" + cacheParam + "' value='" + cacheParam + "' />");
					}
				}
			}
		}

		// Add global action
		i_len = lstOrigin.length;
		for( i = 0; i !== i_len; i += 1 ){
			$origin = lstOrigin[ i ];
			settings = $origin.data( configData );
			if ( settings.action ) {
				lstFieldFlowPostEvent.push( {
					$elm: $origin,
					data: settings
				} );
			}
		}
		
		i_len = lstFieldFlowPostEvent.length;
		for( i = 0; i !== i_len; i += 1 ){
			m_cache = lstFieldFlowPostEvent[ i ];
			m_cache.data.lastProvEvt = lastProvEvt;
			m_cache.$elm.trigger( m_cache.data.action + "." + submitedEvent, m_cache.data );
		}
		
		if ( preventSubmit ) {
			event.preventDefault();
			if ( event.stopPropagation ) {
				event.stopImmediatePropagation();
			} else {
				event.cancelBubble = true;
			}
			return false;
		}
		
	});

	$document.on( "tblfilter." + drawEvent, selector, function( event, data ) {
		if ( event.namespace === drawEvent ) {
			var bindTo = data.bind, // ID of the datatable
				selColumn = data.column, // (integer/datatable column selector)
				csvExtract = data.csvextract, // (true|false) Is we do a data extraction by CVS instead of looking for "li" elements
				$column,
				$datatable = $("#" + bindTo),
				nbSelectedRow,
				arrData, $listItem,
				i, i_len,
				j, j_len,
				items = [ ],
				cur_itm,
				prefLabel = data.label,
				emptyLabel = data.emptylabel,
				lblselector = data.lblselector,
				filterSequence = data.fltrseq ? data.fltrseq : [ ],
				actionItm, filterItm; 

			// Check if the datatable has been loaded, if not we will wait until it is.
			if (! $datatable.hasClass( "wb-tables-inited" ) ) {
				$datatable.one( "wb-ready.wb-tables", function() {
					$(event.target).trigger( "tblfilter." + drawEvent, data );
				} );
				return false;
			}
			$datatable = $datatable.dataTable({ "retrieve": true }).api();

			if ( $datatable.rows( { "search": "applied" } ).data().length <= 1  ) {
				return true;
			}
			
			if ( !selColumn && filterSequence.length ) {
				cur_itm = filterSequence.shift();
				if ( !cur_itm.column ) {
					throw "Column is undefined in the filter sequence";
				}
				selColumn = cur_itm.column;
				csvExtract = cur_itm.csvextract;
				emptyLabel = cur_itm.emptylabel;
				prefLabel = cur_itm.label;
				lblselector = cur_itm.lblselector;
			}
			
			$column = $datatable.column( selColumn, { "search": "applied" } )
			
			// Get the items
			if ( csvExtract ) {
				arrData = $column.data();
				for ( i = 0, i_len = arrData.length; i !== i_len; i += 1 ) {
					items = items.concat( arrData[ i ].split( "," ) );
				}
			} else {
				arrData = $column.nodes();
				for ( i = 0, i_len = arrData.length; i !== i_len; i += 1 ) {
					$listItem = $(arrData[ i ]).find("li");
					for (j = 0, j_len = $listItem.length; j !== j_len; j += 1 ){
						items.push( $( $listItem[ j ] ).text() );
					}
				}
			}
			
			items = items.sort().filter(function(item, pos, ary) {
				return !pos || item != ary[pos - 1];
			});
			
			var elm = event.target,
				$elm = $( elm ),
				itemsToCreate = [ ],
				pushAction = data.actions ? data.actions : [ ];

			if ( filterSequence.length ) {
				filterItm = {
					action: "append", 
					srctype: "tblfilter", 
					bind: bindTo, 
					fltrseq: filterSequence
				};
			}
			for ( i = 0, i_len = items.length; i !== i_len; i += 1 ) {
				cur_itm = items[ i ];
				
				actionItm = {
					label: cur_itm,
					//value: cur_itm,
					actions: [
						{ // Set an action upon item selection
							action: "tblfilter",
							bind: bindTo,
							column: selColumn,
							value: cur_itm
						}
					]
				};
				if ( filterItm ) {
					actionItm.actions.push(filterItm);
				}
				itemsToCreate.push( actionItm );
			}

			if ( lblselector ) {
				prefLabel = $( prefLabel );
			}
			if ( ! prefLabel ) {
				prefLabel = $column.header().textContent;
			}
			
			if ( !data.outputctnrid ) {
				data.outputctnrid = data.provEvt.parentElement.id;
			}
			
			$elm.trigger( "select." + createCtrlEvent, {
				actions: pushAction,
				outputctnrid: data.outputctnrid,
				label: prefLabel,
				emptylabel: emptyLabel,
				lblselector: lblselector,
				items: itemsToCreate
			} );

		}
	});
	
	// Event triggered on the "sub plugin container or the plugin container itself"
	$document.on( componentName + "." + drawEvent, selector, function( event, data ) {
		/*
		 * Data for drawEvent interface
		 *
		drawEvent = {
			container: "", // Container ID where the control will be added
			actions: [ ], // JSON array containing action object to perform upon item selection (Default those action is saved under the "jj-down-push" data property)
			source: "" // Where to take the data [DOM ELEMENT or JQuery selector]
			renderas: "" // How to render the control [select, radio, checkbox]
		}
		*/
		

		if ( event.namespace === drawEvent ) {
			var elm = event.target,
				$elm = $( elm ),
				wbDataElm,
				$source = $( data.source ),
				$labelExplicit, $firstChild,
				labelSelector = "." + labelClass,
				$label = $source.find( "> p" ),
				$items = getItemsData( $source.find( "ul:first() > li" ) ), 
				pushAction, renderas;
			
			// Extend if it is a sub-component
			if ( $source.hasClass( subComponentName ) ){
				wbDataElm = wb.getData( $source, componentName );
				data = $.extend( {}, data, wbDataElm );
			}
			pushAction = data.actions ? data.actions : [ ];
			renderas = data.renderas ? data.renderas : "select"; // Default it will render as select

			// Check if the first node is a div and contain the label.
			$firstChild = $source.children().first();
			
			if ( ! $firstChild.hasClass( headerClass ) ) {
				// Only use what defined as the label, nothing else
				$labelExplicit = $label.find( labelSelector );
				if ( $labelExplicit.length ){
					$label = $labelExplicit;
				}
				$label = $label.html();
				labelSelector = null; // unset the label selector because it not needed for the control creation
			} else {
				$label = $firstChild;
				labelSelector = "." + labelClass;
			}
		
			if ( !data.outputctnrid ) {
				data.outputctnrid = data.provEvt.parentElement.id;
			}
		
			$elm.trigger( renderas + "." + createCtrlEvent, {
				actions: pushAction,
				outputctnrid: data.outputctnrid,
				label: $label,
				lblselector: labelSelector,
				emptylabel: data.emptylabel,
				required: !!!data.notrequired,
				items: $items
				
			} );
		}
	});
	
	function getItemsData ( $items, preventRecusive ) {
		var arrItems = $items.get(),
			i, i_len = arrItems.length, itmCached,
			itmMode, itmLabel, itmValue, itmData, grpItem,
			j, j_len, childNodes, firstNode, childNode, $childNode, childNodeID,
			parsedItms = [],
			actions = [];

		for( i = 0; i !== i_len; i += 1 ){
			itmCached = arrItems[ i ];
			
			itmValue = "";
			grpItem = null;
			itmLabel = "";

			firstNode = itmCached.firstChild;
			childNodes = itmCached.childNodes;
			j_len = childNodes.length;
			
			if (!firstNode ) {
				throw "You have a markup error, There may be an empyt <li> elements in your list.";
			}

			actions = [];
			
			// Is firstNode an anchor?
			if ( firstNode.nodeName === "A" ) {
				// Then we are in redirect mode
				//itmMode = "redirect";
				itmValue = firstNode.getAttribute( "href" );
				itmLabel = $(firstNode).html();
				j_len = 1; // Force following elements to be ignored
				
				actions.push({
					action: "redir",
					url: itmValue
				})
				
			}
		
			// Iterate until we have found the labelClass or <ul> or element with subSelector or end of the array
			for ( j = 1; j !== j_len; j += 1 ){
				childNode = childNodes[ j ];
				$childNode = $( childNode );
				
				// Sub plugin
				if ( $childNode.hasClass( subComponentName ) ) {
					childNodeID = childNode.id || wb.getId();
					childNode.id = childNodeID
					itmValue = componentName + "-" + childNodeID;
					
					actions.push( {
						action: "append",
						srctype: componentName,
						selector: "#" + childNodeID
					} );
					
					break;
				}
				
				// Grouping
				if ( childNode.nodeName === "UL" ) {
					
					if ( preventRecusive ) {
						throw "Recursive error, please check your code";
						return;
					}
					
					// Here a sublist, So the current iterated item are a "group"
					//itmMode = "group";
					grpItem = getItemsData( $childNode.children(), true );
				}
				
				// Explicit label to use
				if ( $childNode.hasClass( labelClass ) ) {
					itmLabel = $childNode.html();
				}
			}
			
			if ( !itmLabel ) {
				itmLabel = firstNode.nodeValue;
			}
			
			// Set an id on the element
			if ( ! itmCached.id ) {
				itmCached.id = wb.getId();
			}

			// Return the item parsed
			parsedItms.push({
				"linkTo": itmCached.id,
				"label": itmLabel,
				"actions": actions,
				"group": grpItem
			});
			
		}
		
		return parsedItms;
	};
	
	$document.on( "select." + createCtrlEvent, selector, function( event, data ) {
		/*
		 * Data structure
		 *
		{
			actions: [ ] // List of action to carry on and to set on the controler...
			outputctnrid, // Where to append the ctrl to.
			label: "", // [html text] Or a jQuery/DOM reference 
			lblselector: "", // (Optional) If set, the label need to be a jQuery/DOM and the inner label will be selected
			emptylabel: "", // (Optional) specified the empty label
			required: false, // Indicator if the control are required
			items: [
				{ 
					label: "",
					value: "", 
					// OR when grouping exist
					value: {
						label: "",
						value: "",
						elmid: ""
					},
					elmid: "" // ID of the element
				}
				
			]
		}*/
		
		var bodyId = data.outputctnrid,
			actions = data.actions,
			label = data.label,
			lblselector = data.lblselector,
			isReq = !!data.required,
			items = data.items,
			elm = event.target,
			$elm = $( elm ),
			i18n = $elm.data( configData ).i18n,
			autoID = wb.getId(),
			labelPrefix = "<label for='" + autoID + "'",
			labelSuffix = "</span>",
			$out, $tmpLabel, 
			selectOut, $selectOut,
			emptyLabel = data.emptylabel ? data.emptylabel : i18n.emptysel,
			i, i_len, j, j_len, cur_itm, cur_itmValue;

		// Create the label
		if ( isReq ) {
			labelPrefix += " class='required'";
			labelSuffix += " <strong class='required'>(" + i18n.required + ")</strong>";
		}
		labelPrefix += "><span class='field-name'>";
		labelSuffix += "</label>";
		if ( ! lblselector ) {
			$out = $(labelPrefix + label + labelSuffix);
		} else {
			$out = label.clone();
			$tmpLabel = $out.find( lblselector );
			$tmpLabel.html( labelPrefix + $tmpLabel.html() + labelSuffix );
		}

		// Create the select
		selectOut = "<select id='" + autoID + "' name='" + basenameInput + autoID + "' class='full-width form-control mrgn-bttm-md " + crtlSelectClass + "' data-" + originData + "='" + elm.id + "'";
		if ( isReq ) {
			selectOut += " required";
		}
		selectOut += "><option value=''>" + emptyLabel + "</option>";
		for ( i = 0, i_len = items.length; i !== i_len; i += 1 ) {
			cur_itm = items[ i ];
			
			
			if ( !cur_itm.group ) {
				selectOut += buildSelectOption( cur_itm );
			} else {
				// We have a group of sub-items
				// The cur_itm are a group
				selectOut += "<optgroup label='" + cur_itm.label + "'>"
				j_len = cur_itm.group.length;
				for ( j = 0; j !== j_len; j += 1 ) {
					selectOut += buildSelectOption( cur_itm.group[ j ] );
				}
				selectOut += "</optgroup>"
			}
		}
		selectOut += '</select>';
		$selectOut = $( selectOut );

		$( "#" + bodyId ).append( $out ).append( $selectOut );

		// Set post action if any
		if ( actions && actions.length > 0 ) {
			$selectOut.data( pushJQData, actions );
		}
		
		// Register this control
		pushData( $elm, registerJQData, autoID );

		
	} );
	
	function buildSelectOption ( data ){
		var label = data.label,
			linkTo = data.linkTo, // Usually an ID
			actions = data.actions,
			out = "<option value='" + label + "'";

		if ( linkTo ) {
			out += " data-" + bindtoData + "='" + linkTo + "'";
		}
		if ( actions ) {
			out += " data-" + actionData + "='" + JSON.stringify(actions) + "'";
		}
		out += ">" + label + "</option>";
		return out;
	};
	
	$document.on( "keyup", selectorForm + " select", function( Ev ) {	
	
		// Add the fix for the on change event - https://bugzilla.mozilla.org/show_bug.cgi?id=126379
		if (navigator.userAgent.indexOf("Gecko") != -1) {
			// prevent tab, alt, ctrl keys from fireing the event
			if (Ev.keyCode && (Ev.keyCode == 1 || Ev.keyCode == 9 || 
				Ev.keyCode == 16 || Ev.altKey || Ev.ctrlKey))
				return true;
			$(Ev.target).trigger("change");
			return true;
		
		}

	});
	
	
	
	$document.on( "checkbox." + createCtrlEvent, selector, function( event, data ) {
		/*
		 * Data structure
		 *
		{
			actions: [ ] // List of action to carry on and to set on the controler...
			outputctnrid, // Where to append the ctrl to.
			label: "", // [html text] Or a jQuery/DOM reference 
			lblselector: "", // (Optional) If set, the label need to be a jQuery/DOM and the inner label will be selected
			emptylabel: "", // (Optional) specified the empty label
			required: false, // Indicator if the control are required
			items: [
				{ 
					label: "",
					value: "", 
					// OR when grouping exist
					value: {
						label: "",
						value: "",
						elmid: ""
					},
					elmid: "" // ID of the element
				}
				
			]
		}*/
		var bodyId = data.outputctnrid,
			actions = data.actions,
			label = data.label,
			lblselector = data.lblselector,
			isReq = !!data.required,
			items = data.items,
			elm = event.target,
			$elm = $( elm ),
			i18n = $elm.data( configData ).i18n,
			ctrlID = wb.getId(),
			fieldsetPrefix = "<legend class='h5 ",
			fieldsetSuffix = "</span>",
			$out = $("<fieldset id='" + ctrlID + "' data-" + originData + "='" + elm.id + "' class='" + crtlSelectClass + " mrgn-bttm-md'></fieldset>"),
			selectOut, $selectOut,
			checkboxOut = "",
			checkboxClass = !data.inline ? "checkbox" : "checkbox-inline",
			fieldName = basenameInput + ctrlID,
			i, i_len, j, j_len, cur_itm;

			
			
		// Create the fieldset
		/*
		<fieldset>
			<legend class="h5">What do you need help with?</legend>
			<lable><input type="checkbox"> Preparing life in Canada</label><br />
			<lable><input type="checkbox"> Working in Canada</label><br />
			<lable><input type="checkbox"> Finding information about French-speaking communities</label>
		</fieldset>*/
		
		// Create the legend
		if ( isReq ) {
			fieldsetPrefix += " required";
			fieldsetSuffix += " <strong class='required'>(" + i18n.required + ")</strong>";
		}
		fieldsetPrefix += "'>";
		fieldsetSuffix += "</legend>";
		if ( ! lblselector ) {
			$out.append( $(fieldsetPrefix + label + fieldsetSuffix) );
		} else {
			$out.append( label.clone() );
			$out.find( lblselector ).html( fieldsetPrefix + $tmpLabel.html() + fieldsetSuffix );
		}
		
		// Create checkbox
		for ( i = 0, i_len = items.length; i !== i_len; i += 1 ) {
			cur_itm = items[ i ];
			
			
			if ( !cur_itm.group ) {
				checkboxOut += buildCheckbox( cur_itm, fieldName, checkboxClass );
			} else {
				// We have a group of sub-items
				// The cur_itm are a group
				checkboxOut += "<p>" + cur_itm.label + "</p>";
				j_len = cur_itm.group.length;
				for ( j = 0; j !== j_len; j += 1 ) {
					checkboxOut += buildCheckbox( cur_itm.group[ j ], fieldName,  checkboxClass );
				}
			}
		}
		$out.append( checkboxOut );
		$( "#" + bodyId ).append( $out );

		// Set post action if any
		if ( actions && actions.length > 0 ) {
			$out.data( pushJQData, actions );
		}
		
		// Register this control
		pushData( $elm, registerJQData, ctrlID );

	} );
	
		
	function buildCheckbox ( data, fieldName, checkboxClass ){
		var label = data.label,
			linkTo = data.linkTo, // Usually an ID
			actions = data.actions,
			fieldID = wb.getId(),
			out = "<div class='" + checkboxClass + "'><label for='" + fieldID + "'><input id='" + fieldID + "' type='checkbox'  name='" + fieldName + "' value='" + label + "'";

		if ( linkTo ) {
			out += " data-" + bindtoData + "='" + linkTo + "'";
		}
		if ( actions ) {
			out += " data-" + actionData + "='" + JSON.stringify(actions) + "'";
		}
		out += " /> " + label + "</label></div>";
		return out;
	};
	
	// Bind the init event of the plugin
	$document.on( "timerpoke.wb " + initEvent, selector, function( event ) {
		var eventTarget = event.target;
		switch ( event.type ) {
			/*
			* Init
			*/
			case "timerpoke":
			case "wb-init":
				init( event );
				break;
		}
	
		/*
		* Since we are working with events we want to ensure that we are being passive about our control,
		* so returning true allows for events to always continue
		*/
		return true;
	});

	// Add the timer poke to initialize the plugin
	wb.add( selector );
})( jQuery, document, wb );
</script>
