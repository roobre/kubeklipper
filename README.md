# kubeklipper

Kubeklipper is a Helm chart allowing you to deploy your 3D printer software stack in Kubernetes.

It includes:
- [Klipper](https://www.klipper3d.org/), a 3D printer firmware that runs on linux hosts and communicates with a think microcontroller counterpart in your 3D printer
- [Moonraker](https://github.com/Arksine/moonraker), a web API for Klipper
- [Mainsail](https://github.com/mainsail-crew/mainsail), a slick web UI for Klipper/Moonraker
- And/or [Fluidd](https://github.com/fluidd-core/fluidd), anoter slick web UI for Klipper/Moonraker
- [µStreamer](https://github.com/pikvm/ustreamer), a lightweight and fast MJPEG-HTTP streamer for your webcam
- [go2rtc](https://github.com/AlexxIT/go2rtc/), a more complex and powerful video streaming solution, as an alternative to ustreamer.
- [Cura](https://ultimaker.com/software/ultimaker-cura/), an advanced slicer software.
- [OrcaSlicer](https://github.com/SoftFever/OrcaSlicer), an slicer as an alternative to Cura.

## Using this chart

The kubeklipper chart is distributed as an [OCI image](https://helm.sh/docs/topics/registries/). You can use it as a dependency if you are using Helm v3.8.0 or newer. To do so, simply add an entry to your `dependencies` array like the following:

```yaml
dependencies:
  - name: kubeklipper
    version: 0.1.0
    repository: oci://ghcr.io/roobre/charts
```

You can also install the chart directly:

```shell
helm install kubeklipper oci://ghcr.io/roobre/charts/kubeklipper --version 0.1.0
```

Things you will be able to do _the Kubernetes way_:

- Run a standard linux distro and kubelet on whatever compute you have connected to your 3D printer.
- Store your klipper, moonraker, and mainsail configs as-code, in your `values.yaml` file.
- Distribute other components (web UI, slicer) across different nodes.
- Leverage Kubernetes Ingress objects for local/remote access and TLS termination.
- Upgrade all* your 3D printing software at once, using gitops (if you want to).

Things you will need to do _the old way_:

- Flashing klipper on your 3D printer has to be done manually, as a prerequisite (*).
  - The `mkuf/klipper:latest-tools` image from the prind project contains the toolchain needed for this.
  - As klipper firmware does not change often, you won't need to do this more than once a year.
  - Have any ideas on better ways to do this? Let me know on an issue!
- Configure Cura using its GUI. Cura config is extensive and the software relies on having write access to it.

Things the chart doesn't currently do but might in the future:

- Better volume management. Currently users are allowed to template a `volume` spec, but not to create PVs or PVCs.
  - You can still create those outside of the chart and reference the volumes to them.

## Architecture

Klipper and Moonraker run together, as different containers, in the same pod. This allows them to communicate over a unix socket. You should place node selector, affinities, or resources accordingly so the node running this pod can communicate with your 3D printer via USB.

Mainsail and/or Fluidd run each one on its own pod. All of them communicate with the Klipper/Moonraker pod over HTTP, so they do not necessarily need to run on a node connected to the printer.

uStreamer also runs as an independent pod, but like Klipper/Moonraker, you should specify selectors, affinities, or resources to pin it to the node with the required webcam.

Cura also runs as an independent pod.

### Note on `smarter-device-manager`

I recommend [smarter-device-manager](https://gitlab.com/arm-research/smarter/smarter-device-manager) to allow pods to access device nodes (`/dev/*`), as it provides a secure way to map only certain devices to the pods, while also removing the need to set nodeSelectors or affinities.

## Images

This chart uses some of the awesome images built for the [prind](https://github.com/mkuf/prind) project, as well as the [docker-cura](https://github.com/linuxserver/docker-cura) image maintained by the excellent linuxserver folks.
