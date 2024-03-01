---
title: 2023-12-19 - Emergency
pcx_content_type: changelog
weight: 27772
layout: wide
---

# 2023-12-19 - Emergency

{{<table-wrap>}}
<table style="width: 100%">
  <thead>
    <tr>
      <th>Rule ID</th>
      <th>Description</th>
      <th>Previous Action</th>
      <th>New Action</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>...1fc1e601</td>
      <td>HTTP requests with unusual HTTP headers or URI path (signature #31).</td>
      <td>block</td>
      <td>block</td>
      <td>Add more characteristics to the unusual HTTP headers or URI path.</td>
    </tr>
<tr>
      <td>...22807318</td>
      <td>HTTP requests from known botnets.</td>
      <td>log</td>
      <td>ddos_dynamic</td>
      <td>Extend the rule to catch more attacks.</td>
    </tr>
<tr>
      <td>...d2f294d7</td>
      <td>HTTP requests trying to impersonate browsers.</td>
      <td>ddos_dynamic</td>
      <td>ddos_dynamic</td>
      <td>Change the rule to catch more attacks.</td>
    </tr>
  </tbody>
</table>
{{</table-wrap>}}