---
layout: raw-html
---
<html>
    <head>
        <title data-bind="text: name"></title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/knockout/3.5.0/knockout-min.js"></script>
        <script src="https:////cdn.datatables.net/1.10.20/js/jquery.dataTables.min.js"></script>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
        <link rel="stylesheet" href="https://cdn.datatables.net/1.10.20/css/jquery.dataTables.min.css" />
        <style>
            #schedule table {
                text-align: left;
            }
            tr.even-row {
                background-color: #e0e0e0 !important;
            }
            tr.odd-row {
            }
            #summary > div {
                display: flex;
            }
            #summary ul {
                display: block;
                list-style-type: none;
            }
        </style>
    </head>
    <body>
        <div id="summary">
            <h2 data-bind="text: name"></h2>
            <div>
                <div>
                    <table>
                        <tr>
                            <td>W-L:</td>
                            <td data-bind="text: winLossRecord"></td>
                        </tr>
                        <tr>
                            <td>Win %:</td>
                            <td data-bind="text: winPercentage"></td>
                        </tr>
                        <tr>
                            <td>National Rank:</td>
                            <td data-bind="text: nationalRank"></td>
                        </tr>
                        <tr>
                            <td>State Rank:</td>
                            <td data-bind="text: stateRank"></td>
                        </tr>
                        <tr>
                            <td>Class:</td>
                            <td data-bind="text: stateClass"></td>
                        </tr>
                    </table>
                </div>
                <div>
                    <ul>
                        <li>
                            <a data-bind="attr: {href: boysTeamUrl}, text: 'Boys'"></a>
                        <li>
                        </li>
                            <a data-bind="attr: {href: girlsTeamUrl}, text: 'Girls'"></a>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
        <div id="schedule">
            <table>
                <thead>
                    <tr>
                        <th>Date</th>
                        <th>Name</th>
                        <th>Result</th>
                        <th>Pts Scored</th>
                        <th>Pts Allowed</th>
                        <th>Margin</th>
                        <th>Class</th>
                        <th>W-L</th>
                        <th>Win %</th>
                        <th>National Rank</th>
                        <th>State Rank</th>
                        <th>Stats</th>
                    </tr>
                </thead>
                <tbody data-bind="foreach: games">
                    <tr>
                        <td data-bind="attr: {'data-sort': gameIndex}">
                            <!-- ko if: hasBeenPlayed -->
                                <a data-bind="text: date, attr: {href: gameLink}"></a>
                            <!-- /ko -->
                            <!-- ko ifnot: hasBeenPlayed -->
                                <span data-bind="text: date"></span>
                            <!-- /ko -->
                        </td>
                        <td>
                            <a data-bind="text: nameAndLocation, attr: {href: opponent.teamUrl}"></a>
                        </td>
                        <td data-bind="text: winText"></td>
                        <td data-bind="text: pointsScored"></td>
                        <td data-bind="text: pointsAllowed"></td>
                        <td data-bind="text: pointMargin"></td>
                        <td data-bind="text: opponent.stateClass"></td>
                        <td data-bind="text: opponent.winLossRecord"></td>
                        <td data-bind="text: opponent.winPercentage"></td>
                        <td data-bind="text: opponent.nationalRank"></td>
                        <td data-bind="text: opponent.stateRank"></td>
                        <td>
                            <a data-bind="text: 'Stats', attr: {href: opponent.statsUrl}"></a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </body>
    <script type="text/javascript">

        class TeamInfo {
            constructor(maxPrepsTeamId, season, loadScheduleTeamInfo){
                this.maxPrepsTeamId = maxPrepsTeamId;
                this.season = season;
                this.loadScheduleTeamInfo = loadScheduleTeamInfo;
            }
            games = [];

            name = "Unknown";
            stateClass = "";
            winLossRecord = "";
            nationalRank = "";
            stateRank = "";
            statsUrl = "";
            teamUrl = "";
            girlsTeamUrl = "";
            boysTeamUrl = "";

            get winPercentage() {
                let split = this.winLossRecord.split("-");
                if(split.length == 2){
                    let wins = parseInt(split[0]);
                    let losses = parseInt(split[1]);
                    let winPercentage = 100 * wins / (wins + losses);
                    return winPercentage.toFixed(0) + "%";
                }
                return "";
            }

            get sportId() {
                if(this.season == null){
                    return "basketball";
                }else{
                    return `basketball-winter-${this.season}`;
                }
            }

            request(obj) {
                return new Promise((resolve, reject) => {
                    let xhr = new XMLHttpRequest();
                    xhr.open(obj.method || "GET", obj.url);
                    xhr.responseType = "document";
                    if (obj.headers) {
                        Object.keys(obj.headers).forEach(key => {
                            xhr.setRequestHeader(key, obj.headers[key]);
                        });
                    }
                    xhr.onload = () => {
                        if (xhr.status >= 200 && xhr.status < 300) {
                            resolve(xhr.responseXML);
                        } else {
                            reject(xhr.statusText);
                        }
                    };
                    xhr.onerror = () => reject(xhr.statusText);
                    xhr.send(obj.body);
                });
            };

            async load(){
                try
                {
                    this.scheduleUrl = `https://preps.origas.org/high-schools/${this.maxPrepsTeamId}/${this.season}/schedule.htm`;
                    let response = await this.request({url: this.scheduleUrl});
                    await this.parse(response);
                }
                catch(e)
                {
                    console.error(e);
                }
            }

            async parse(xml){
                this.doc = xml;

                this.bodyElement = $(this.doc);

                this.name = "Unknown";
                let jsonTags = this.bodyElement.find("script[type='application/ld+json']");
                if(jsonTags.length > 0){
                    let json = jsonTags[0].innerText;
                    let embeddedInfo = JSON.parse(json)
                    this.name = embeddedInfo.name.replace(" High", "").replace(" School", "");
                }

                this.stateClass = "";
                let districtLink = this.bodyElement.find("a[href^='/league']");
                if(districtLink.length > 0){
                    let href = districtLink.attr("href");
                    let urlChunks = href.split("/");
                    let districtChunk = urlChunks[urlChunks.length - 1];
                    let districtLinkSegments = districtChunk.split("-")
                    if(districtLinkSegments.length >= 2){
                        let possibleClassName = districtLinkSegments[1];
                        if(possibleClassName.length == 2){
                            this.stateClass = possibleClassName.toUpperCase();
                        }
                    }
                }

                this.winLossRecord = this.parseTextFromSelector("#ctl00_NavigationWithContentOverRelated_ContentOverRelated_PageHeaderUserControl_BottomRowOverallRecord > a");

                this.nationalRank = this.parseTextFromSelector("a[href$='national_rankings']");

                this.stateRank = this.parseTextFromSelector("a[href$='state_rankings']");

                this.statsUrl = `https://preps.origas.org/high-schools/${this.maxPrepsTeamId}/${this.season}/stats.htm`;

                let urlSplit = window.location.href.split("?");
                let noParamsUrl = urlSplit[0];
                let params = urlSplit.length > 1 ? urlSplit[1] : "";

                let urlSearch = new URLSearchParams(params);
                urlSearch.set("teamId", this.maxPrepsTeamId);
                if(this.season != null){
                    urlSearch.set("season", this.season);
                }
                this.teamUrl = noParamsUrl + "?" + urlSearch.toString();

                let girlsSeasonStart = "girls-";
                if(this.season.indexOf(girlsSeasonStart) >= 0){
                    this.girlsTeamUrl = this.teamUrl;
                    let boysSeason = this.season.substring(girlsSeasonStart.length);
                    urlSearch.set("season", boysSeason);
                    this.boysTeamUrl = noParamsUrl + "?" + urlSearch.toString();
                }else{
                    this.boysTeamUrl = this.teamUrl;
                    let girlsSeason = girlsSeasonStart + this.season;
                    urlSearch.set("season", girlsSeason);
                    this.girlsTeamUrl = noParamsUrl + "?" + urlSearch.toString();
                }

                // this.games = [];
                if(this.loadScheduleTeamInfo){
                    let scheduleElement = this.doc.getElementById("schedule");
                    let scheduleRows = scheduleElement.getElementsByClassName("dual-contest");
                    let loadingPromises = [];
                    for(var i = 0; i < scheduleRows.length; i++){
                        let loadingPromise = this.loadGame(scheduleRows[i], i);
                        loadingPromises.push(loadingPromise)
                    }
                    await Promise.all(loadingPromises);
                }
            }

            async loadGame(gameRow, i){
                let game = new Game(gameRow, i, this.season);
                if(game.opponentId != null){
                    this.games.push(game);
                    await game.load();
                }
            }

            parseTextFromSelector(selector){
                let found = this.bodyElement.find(selector);
                if(found.length > 0){
                    return found[0].innerText;
                }
                return "";
            }
        }

        class Game{
            constructor(gameRow, gameIndex, season){
                this.gameRow = gameRow;
                this.gameIndex = gameIndex;
                this.season = season;
                this.isWin = false;
                this.score = "";
                this.pointsScored = "";
                this.pointsAllowed = "";
                this.pointMargin = "";
                this.gameLink = "";
                this.hasBeenPlayed = false;
                this.opponentId = null;


                this.date = this.parseTextFromFirstClassItem("event-date");
                this.isHome = gameRow.getElementsByClassName("away-indicator").length == 0;
                let gameElements = gameRow.getElementsByClassName("score");

                if(gameElements.length > 0){
                    let gameElement = gameElements[0];
                    let scoreText = gameElement.innerText;
                    this.isWin = scoreText.indexOf("W") >= 0;
                    this.score = scoreText.substring(4);
                    let scoreSplit = this.score.split("-");
                    this.pointsScored = parseInt(this.isWin ? scoreSplit[0] : scoreSplit[1]);
                    this.pointsAllowed = parseInt(this.isWin ? scoreSplit[1] : scoreSplit[0]);
                    this.pointMargin = this.pointsScored - this.pointsAllowed;
                    let rawGameLink = gameElement.getAttribute("href");
                    this.gameLink = rawGameLink.replace("www.maxpreps.com", "preps.origas.org");
                    this.hasBeenPlayed = true;
                }

                let opponentLinkElement = gameRow.getElementsByClassName("contest-type-indicator");
                if(opponentLinkElement.length > 0){
                    let link = opponentLinkElement[0].getAttribute("href");
                    if(link != null){
                        let splitLink = link.split("/");
                        if(splitLink.length >= 3){
                            this.opponentId = splitLink[2];
                        }
                    }
                }
            }

            get winText() {
                if(this.hasBeenPlayed){
                    return this.isWin ? "W" : "L";
                }else{
                    return "";
                }
            }

            get nameAndLocation() {
                return `${this.isHome ? "" : "@"} ${this.opponent.name}`;
            }

            async load(){
                this.opponent = new TeamInfo(this.opponentId, this.season, false);
                await this.opponent.load();
            }
            parseTextFromFirstClassItem(className){
                let found = this.gameRow.getElementsByClassName(className);
                if(found.length > 0){
                    return found[0].innerText;
                }
                return "";
            }
        }

        $(async function(){
            let urlSearch = new URLSearchParams(window.location.search);
            let teamId = urlSearch.get("teamId");
            if(teamId == null){
                teamId = "college-view-academy-eagles-(lincoln,ne)";
            }
            let season = urlSearch.get("season");
            if(season == null){
                let date = new Date();
                let year = date.getFullYear() - 2000;
                if(date.getMonth() >= 6){
                    year += 1;
                }
                season = `basketball-winter-${year-1}-${year}`;
            }
            let team = new TeamInfo(teamId, season, true);
            await team.load();
            ko.applyBindings(team);
            $("#schedule > table").DataTable({
                paging: false, 
                searching: false,
                info: false,
                stripeClasses: ['odd-row', 'even-row']
            });
            window.team = team;
        });
    </script>
</html>