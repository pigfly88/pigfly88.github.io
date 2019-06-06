- 1xx: Informational - Request received, continuing process
- 2xx: Success - The action was successfully received, understood, and accepted
- 3xx: Redirection - Further action must be taken in order to complete the request
- 4xx: Client Error - The request contains bad syntax or cannot be fulfilled
- 5xx: Server Error - The server failed to fulfill an apparently valid request

<table>
      <thead>
        <tr style="cursor: pointer;">
          <th>Value</th>
          <th>Description</th>
          <th>Reference</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td align="center">100</td>
          <td>Continue</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.2.1</a>]</td>
        </tr>
        <tr>
          <td align="center">101</td>
          <td>Switching Protocols</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.2.2</a>]</td>
        </tr>
        <tr>
          <td align="center">102</td>
          <td>Processing</td>
          <td>[<a href="http://www.iana.org/go/rfc2518">RFC2518</a>]</td>
        </tr>
        <tr>
          <td align="center">103</td>
          <td>Early Hints</td>
          <td>[<a href="http://www.iana.org/go/rfc8297">RFC8297</a>]</td>
        </tr>
        <tr>
          <td align="center">104-199</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">200</td>
          <td>OK</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.3.1</a>]</td>
        </tr>
        <tr>
          <td align="center">201</td>
          <td>Created</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.3.2</a>]</td>
        </tr>
        <tr>
          <td align="center">202</td>
          <td>Accepted</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.3.3</a>]</td>
        </tr>
        <tr>
          <td align="center">203</td>
          <td>Non-Authoritative Information</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.3.4</a>]</td>
        </tr>
        <tr>
          <td align="center">204</td>
          <td>No Content</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.3.5</a>]</td>
        </tr>
        <tr>
          <td align="center">205</td>
          <td>Reset Content</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.3.6</a>]</td>
        </tr>
        <tr>
          <td align="center">206</td>
          <td>Partial Content</td>
          <td>[<a href="http://www.iana.org/go/rfc7233">RFC7233, Section 4.1</a>]</td>
        </tr>
        <tr>
          <td align="center">207</td>
          <td>Multi-Status</td>
          <td>[<a href="http://www.iana.org/go/rfc4918">RFC4918</a>]</td>
        </tr>
        <tr>
          <td align="center">208</td>
          <td>Already Reported</td>
          <td>[<a href="http://www.iana.org/go/rfc5842">RFC5842</a>]</td>
        </tr>
        <tr>
          <td align="center">209-225</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">226</td>
          <td>IM Used</td>
          <td>[<a href="http://www.iana.org/go/rfc3229">RFC3229</a>]</td>
        </tr>
        <tr>
          <td align="center">227-299</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">300</td>
          <td>Multiple Choices</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.4.1</a>]</td>
        </tr>
        <tr>
          <td align="center">301</td>
          <td>Moved Permanently</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.4.2</a>]</td>
        </tr>
        <tr>
          <td align="center">302</td>
          <td>Found</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.4.3</a>]</td>
        </tr>
        <tr>
          <td align="center">303</td>
          <td>See Other</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.4.4</a>]</td>
        </tr>
        <tr>
          <td align="center">304</td>
          <td>Not Modified</td>
          <td>[<a href="http://www.iana.org/go/rfc7232">RFC7232, Section 4.1</a>]</td>
        </tr>
        <tr>
          <td align="center">305</td>
          <td>Use Proxy</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.4.5</a>]</td>
        </tr>
        <tr>
          <td align="center">306</td>
          <td>(Unused)</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.4.6</a>]</td>
        </tr>
        <tr>
          <td align="center">307</td>
          <td>Temporary Redirect</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.4.7</a>]</td>
        </tr>
        <tr>
          <td align="center">308</td>
          <td>Permanent Redirect</td>
          <td>[<a href="http://www.iana.org/go/rfc7538">RFC7538</a>]</td>
        </tr>
        <tr>
          <td align="center">309-399</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">400</td>
          <td>Bad Request</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.1</a>]</td>
        </tr>
        <tr>
          <td align="center">401</td>
          <td>Unauthorized</td>
          <td>[<a href="http://www.iana.org/go/rfc7235">RFC7235, Section 3.1</a>]</td>
        </tr>
        <tr>
          <td align="center">402</td>
          <td>Payment Required</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.2</a>]</td>
        </tr>
        <tr>
          <td align="center">403</td>
          <td>Forbidden</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.3</a>]</td>
        </tr>
        <tr>
          <td align="center">404</td>
          <td>Not Found</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.4</a>]</td>
        </tr>
        <tr>
          <td align="center">405</td>
          <td>Method Not Allowed</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.5</a>]</td>
        </tr>
        <tr>
          <td align="center">406</td>
          <td>Not Acceptable</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.6</a>]</td>
        </tr>
        <tr>
          <td align="center">407</td>
          <td>Proxy Authentication Required</td>
          <td>[<a href="http://www.iana.org/go/rfc7235">RFC7235, Section 3.2</a>]</td>
        </tr>
        <tr>
          <td align="center">408</td>
          <td>Request Timeout</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.7</a>]</td>
        </tr>
        <tr>
          <td align="center">409</td>
          <td>Conflict</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.8</a>]</td>
        </tr>
        <tr>
          <td align="center">410</td>
          <td>Gone</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.9</a>]</td>
        </tr>
        <tr>
          <td align="center">411</td>
          <td>Length Required</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.10</a>]</td>
        </tr>
        <tr>
          <td align="center">412</td>
          <td>Precondition Failed</td>
          <td>[<a href="http://www.iana.org/go/rfc7232">RFC7232, Section 4.2</a>][<a href="http://www.iana.org/go/rfc8144">RFC8144, Section 3.2</a>]</td>
        </tr>
        <tr>
          <td align="center">413</td>
          <td>Payload Too Large</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.11</a>]</td>
        </tr>
        <tr>
          <td align="center">414</td>
          <td>URI Too Long</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.12</a>]</td>
        </tr>
        <tr>
          <td align="center">415</td>
          <td>Unsupported Media Type</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.13</a>][<a href="http://www.iana.org/go/rfc7694">RFC7694, Section 3</a>]</td>
        </tr>
        <tr>
          <td align="center">416</td>
          <td>Range Not Satisfiable</td>
          <td>[<a href="http://www.iana.org/go/rfc7233">RFC7233, Section 4.4</a>]</td>
        </tr>
        <tr>
          <td align="center">417</td>
          <td>Expectation Failed</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.14</a>]</td>
        </tr>
        <tr>
          <td align="center">418-420</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">421</td>
          <td>Misdirected Request</td>
          <td>[<a href="http://www.iana.org/go/rfc7540">RFC7540, Section 9.1.2</a>]</td>
        </tr>
        <tr>
          <td align="center">422</td>
          <td>Unprocessable Entity</td>
          <td>[<a href="http://www.iana.org/go/rfc4918">RFC4918</a>]</td>
        </tr>
        <tr>
          <td align="center">423</td>
          <td>Locked</td>
          <td>[<a href="http://www.iana.org/go/rfc4918">RFC4918</a>]</td>
        </tr>
        <tr>
          <td align="center">424</td>
          <td>Failed Dependency</td>
          <td>[<a href="http://www.iana.org/go/rfc4918">RFC4918</a>]</td>
        </tr>
        <tr>
          <td align="center">425</td>
          <td>Too Early</td>
          <td>[<a href="http://www.iana.org/go/rfc8470">RFC8470</a>]</td>
        </tr>
        <tr>
          <td align="center">426</td>
          <td>Upgrade Required</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.5.15</a>]</td>
        </tr>
        <tr>
          <td align="center">427</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">428</td>
          <td>Precondition Required</td>
          <td>[<a href="http://www.iana.org/go/rfc6585">RFC6585</a>]</td>
        </tr>
        <tr>
          <td align="center">429</td>
          <td>Too Many Requests</td>
          <td>[<a href="http://www.iana.org/go/rfc6585">RFC6585</a>]</td>
        </tr>
        <tr>
          <td align="center">430</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">431</td>
          <td>Request Header Fields Too Large</td>
          <td>[<a href="http://www.iana.org/go/rfc6585">RFC6585</a>]</td>
        </tr>
        <tr>
          <td align="center">432-450</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">451</td>
          <td>Unavailable For Legal Reasons</td>
          <td>[<a href="http://www.iana.org/go/rfc7725">RFC7725</a>]</td>
        </tr>
        <tr>
          <td align="center">452-499</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">500</td>
          <td>Internal Server Error</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.6.1</a>]</td>
        </tr>
        <tr>
          <td align="center">501</td>
          <td>Not Implemented</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.6.2</a>]</td>
        </tr>
        <tr>
          <td align="center">502</td>
          <td>Bad Gateway</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.6.3</a>]</td>
        </tr>
        <tr>
          <td align="center">503</td>
          <td>Service Unavailable</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.6.4</a>]</td>
        </tr>
        <tr>
          <td align="center">504</td>
          <td>Gateway Timeout</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.6.5</a>]</td>
        </tr>
        <tr>
          <td align="center">505</td>
          <td>HTTP Version Not Supported</td>
          <td>[<a href="http://www.iana.org/go/rfc7231">RFC7231, Section 6.6.6</a>]</td>
        </tr>
        <tr>
          <td align="center">506</td>
          <td>Variant Also Negotiates</td>
          <td>[<a href="http://www.iana.org/go/rfc2295">RFC2295</a>]</td>
        </tr>
        <tr>
          <td align="center">507</td>
          <td>Insufficient Storage</td>
          <td>[<a href="http://www.iana.org/go/rfc4918">RFC4918</a>]</td>
        </tr>
        <tr>
          <td align="center">508</td>
          <td>Loop Detected</td>
          <td>[<a href="http://www.iana.org/go/rfc5842">RFC5842</a>]</td>
        </tr>
        <tr>
          <td align="center">509</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
        <tr>
          <td align="center">510</td>
          <td>Not Extended</td>
          <td>[<a href="http://www.iana.org/go/rfc2774">RFC2774</a>]</td>
        </tr>
        <tr>
          <td align="center">511</td>
          <td>Network Authentication Required</td>
          <td>[<a href="http://www.iana.org/go/rfc6585">RFC6585</a>]</td>
        </tr>
        <tr>
          <td align="center">512-599</td>
          <td>Unassigned</td>
          <td></td>
        </tr>
      </tbody>
    </table>

### 参考资料
- [https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)
- [https://tools.ietf.org/html/rfc7231#page-47](https://tools.ietf.org/html/rfc7231#page-47)
- [https://en.wikipedia.org/wiki/List_of_HTTP_status_codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
- [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)