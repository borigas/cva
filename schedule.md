---
layout: raw-html
---
<html>
    <head>
        <title>Schedule</title>
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
                            <td>Division Rank:</td>
                            <td data-bind="text: divisionRank"></td>
                        </tr>
                        <tr>
                            <td>League:</td>
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
                        <th>Division Rank</th>
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
                        <td data-bind="text: opponent.divisionRank"></td>
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
            divisionRank = "";
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
                    this.infoUrl = `https://preps.origas.org/high-schools/${this.maxPrepsTeamId}/${this.season}/info.htm`;
                    let infoPromise = await this.request({url: this.infoUrl});
                    let infoResponse = await infoPromise;

                    await this.parse(infoResponse);
                }
                catch(e)
                {
                    console.error(e);
                }
            }

            async parse(infoXml){
                this.infoDoc = infoXml;
                this.infoBodyElement = $(this.infoDoc);

                this.name = "Unknown";
                let headers = this.infoDoc.getElementsByTagName("h1");
                if(headers.length > 0){
                    let headerText = headers[0].innerText;
                    this.name = headerText.replace(" High", "").replace(" School", "").replace(" Basketball", "").replace(" Boys", "").replace(" Basketball Team Info", "").replace(/[0-9]/g, "").trim();
                }

                this.stateClass = "";
                let rankHeadings = this.infoBodyElement.find(".Anchor__StyledAnchor-sc-1fyvlgd-0.gMjAj");
                if(rankHeadings.length == 3){
                    this.stateClass = rankHeadings[1].innerText.replace(" Division", "");
                }

                this.winLossRecord = this.parseTextFromSelector(this.infoBodyElement, ".Text__StyledText-jknly0-0.PiNEz")

                /*
                let ranks = this.infoBodyElement.find(".Text__StyledText-jknly0-0.iMvTHW");
                if(ranks.length > 1){
                    this.stateRank = ranks[0].innerText.replace("#", "");

                    if(ranks.length > 2){
                        this.nationalRank = ranks[2].innerText.replace("#", "");
                        this.divisionRank = ranks[1].innerText.replace("#", "");
                    }else{
                        this.nationalRank = ranks[1].innerText.replace("#", "");
                        this.divisionRank = "";
                    }
                }
                */

                var xpath = "//span[contains(text(),'#')]";
                var xpathResults = this.infoDoc.evaluate(xpath, this.infoDoc, null, XPathResult.ANY_TYPE, null)
                let rankNumbers = [];
                var rankNode;
                while(rankNode = xpathResults.iterateNext()){
                    rankNumbers.push(rankNode.innerText.replace("#",""));
                }
                if(rankNumbers.length > 1){
                    this.stateRank = rankNumbers[0];

                    if(rankNumbers.length > 2){
                        this.nationalRank = rankNumbers[2];
                        this.divisionRank = rankNumbers[1];
                    }else{
                        this.nationalRank = rankNumbers[1];
                        this.divisionRank = "";
                    }
                }

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

                if(this.loadScheduleTeamInfo){
                    let scheduleUrl = `https://preps.origas.org/high-schools/${this.maxPrepsTeamId}/${this.season}/schedule.htm`;
                    let schedulePromise = this.request({url: scheduleUrl});
                    let scheduleResponse = await schedulePromise;
                    let scheduleElement = scheduleResponse.getElementsByTagName("tbody")[0];
                    let scheduleRows = scheduleElement.getElementsByTagName("tr");
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

            parseTextFromSelectorLast(bodyElement, selector){
                let found = bodyElement.find(selector);
                if(found.length > 0){
                    return found[found.length - 1].innerText;
                }
                return "";
            }

            parseTextFromSelectorIndex(bodyElement, selector, index){
                let found = bodyElement.find(selector);
                if(found.length > index){
                    return found[index].innerText;
                }
                return "";
            }

            parseTextFromSelector(bodyElement, selector){
                return this.parseTextFromSelectorIndex(bodyElement, selector, 0);
            }

            parseText
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
                this.date = "";

                let gameCells = this.gameRow.getElementsByTagName("td");
                if(gameCells.length < 4){
                    return;
                }

                let dateCell = gameCells[0];
                let gameDateElements = dateCell.getElementsByTagName("div");
                if(gameDateElements.length > 0){
                    let dateText = gameDateElements[0].innerText;
                    this.date = dateText.split(", ")[1];
                }

                let opponentCell = gameCells[2];
                this.isHome = opponentCell.innerText.indexOf("Home") >= 0;
                let opponentLinkElements = gameRow.getElementsByTagName("a");
                if(opponentLinkElements.length > 0){
                    let link = opponentLinkElements[0].getAttribute("href");
                    if(link != null){
                        let splitLink = link.split("/");
                        if(splitLink.length >= 3){
                            this.opponentId = splitLink[4];
                        }
                    }
                }

                let resultCell = gameCells[3];
                var resultSpans = resultCell.getElementsByTagName("span");
                if(resultSpans.length >= 2){
                    this.isWin = resultSpans[0].innerText.indexOf("W") >= 0;
                    this.score = resultSpans[1].innerText;
                    let scoreSplit = this.score.split("-");
                    this.pointsScored = parseInt(this.isWin ? scoreSplit[0] : scoreSplit[1]);
                    this.pointsAllowed = parseInt(this.isWin ? scoreSplit[1] : scoreSplit[0]);
                    this.pointMargin = this.pointsScored - this.pointsAllowed;
                    var gameLinks = resultCell.getElementsByTagName("a");
                    if(gameLinks.length > 0){
                        this.gameLink = "https://preps.origas.org/" + gameLinks[0].getAttribute("href");
                    }
                    this.hasBeenPlayed = true;
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
            document.title = team.name + " Schedule";
        });
    </script>
</html>