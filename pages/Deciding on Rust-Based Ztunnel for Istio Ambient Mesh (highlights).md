title:: Deciding on Rust-Based Ztunnel for Istio Ambient Mesh (highlights)
author:: [[solo.io]]
full-title:: "Deciding on Rust-Based Ztunnel for Istio Ambient Mesh"
category:: #articles
url:: https://www.solo.io/blog/rust-ztunnel-istio-ambient-mesh/
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/ogimage.php)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- We don’t talk about this often, but if you go through the Envoy configuration for Envoy-based ztunnel, you’ll see tens of thousands of lines for just 2 or 3 services added to your ambient mesh. Envoy wasn’t designed for multi-tenancy, however, ztunnel is multi-tenant and manages all the incoming and outgoing traffic for all co-located pods on the same node. ([View Highlight](https://read.readwise.io/read/01hv1yx4sv0hbrmhj9jj17m0wf))