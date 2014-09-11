docker-cftesting
================

Docker container based on finalcut/mxunit that also provides varscoper at /tmp/varscoper

[varscoper](http://varscoper.riaforge.org/) was authored by Mike Schierberl.  The version included in this docker container is 1.3 (updated Dec 9, 2010).

mxunit is exposed at /tmp/mxunit
varscoper at /tmp/varscoper


This exists basically because my CF builds typically also run a varscoper test that utilizes the power of mxunit and varscoper combined.

```cfm
<cfcomponent name="unittests.utilities.varscoper"  extends="unittests.BaseTest" output="false">

	<cffunction name="varScoper"  access="public" output="false">
		<cfset local = structNew() />
		<cfset var files = "" />

		<cfset local.resultsDir = getConfigService().getProperty("rootpath") & "testresults\varscoper\" />

		<cfif DirectoryExists(local.resultsDir)>
			<cfdirectory action="list" directory="#local.resultsDir#" name="files">
			<cfif files.recordCount>
				<cfquery name="files" dbtype="query">
					SELECT * FROM files ORDER BY datelastmodified desc
				</cfquery>

				<cffile action="read" file="#files.directory#\#files.name#" variable="local.file" />
				<cfset local.file = trim(local.file) />
				<cfif ListLen(local.file, "#CHR(13)#") GT 1>
					<cfset local.msg = "Var Scoping missing in the following file(s) <ul>"/>
					<cfloop from="2" to="#ListLen(local.file, CHR(13))#" index="local.i">
						<cfset local.line = listGetAt(local.file, local.i, CHR(13)) />
						<cftry>
						<cfset  local.msg = local.msg & "<li>" & listGetAt(local.line, 1) & " at line " & ListGetAt(local.line, 3) & " for variable '" & ListGetAt(local.line, 4) &"'</li>" />
						<cfcatch>

						</cfcatch>
						</cftry>
					</cfloop>
					<cfset local.msg = local.msg & "</ul>" />
				<cfset fail(local.msg) />
				</cfif>
			</cfif>
		</cfif>

	</cffunction>


</cfcomponent>
```

This basically goes through the varscoper results and builds an mxunit failure message from the results so the build will include this as a failed test along with details on where the var scoping is missing.
