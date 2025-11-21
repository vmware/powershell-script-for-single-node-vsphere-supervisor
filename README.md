# VCF Edge - Single-Node vSphere Supervisor with ArgoCD Automation Script

This Powershell automation script is designed to streamline the deployment of a single-node vSphere Supervisor in VMware Cloud Foundation (VCF) 9.0 environments based on this [design guidance](https://blogs.vmware.com/cloud-foundation/2025/07/14/modernizing-your-edge-with-single-node-vsphere-supervisor-in-vmware-cloud-foundation-9-0/) It automates critical steps from initial setup to the creation of the supervisor, including network configuration and content library verification. The script leverages VCF.PowerCLI cmdlets and relies on pre-configured JSON input files (infrastructure.json and supervisor.json) to tailor the deployment to your environment. Additionally, it addresses the prerequisites for integrating Argo CD operator services and ensures the necessary CLI tools are available for a comprehensive, end-to-end automated solution.


# SOFTWARE LICENSE AGREEMENT

Copyright (c) CA, Inc. All rights reserved.

You are hereby granted a non-exclusive, worldwide, royalty-free license under CA, Inc.’s copyrights to use, copy, modify, and distribute this software in source code or binary form for use in connection with CA, Inc. products.

This copyright notice shall be included in all copies or substantial portions of the software.

**THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.**

