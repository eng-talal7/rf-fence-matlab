# Documentation review record

Review date: 2026-11-03

## Provenance and scope

- Project: the team's RF Fence MATLAB simulation; a teammate previously hosted a copy of the shared files.
- Reviewed baseline commit: `63ff1d95cfb5edb1a13c80c9e4cdee657f7cd67c`
- Source tree: `fed363c44e32c38df0408b4a70cbfea6fa08a127`
- Destination: https://github.com/eng-talal7/rf-fence-matlab
- Repository copy retained the project files and existing history. The account that hosted the earlier copy does not establish authorship or individual project responsibilities.

All 14 source files were read: both README files and all 12 MATLAB files. The review covered the physics interfaces, coordinate conventions, channel equations, detector protocol, test definitions, dashboard logic and generated-file dependencies. This was source inspection, not numerical certification.

## Findings reflected in README

1. The old README documented only seven physics files; it omitted the channel, detector comparison and dashboard now present.
2. The physical model is scalar diffraction from a flat rectangular screen. The enlarged drone icon is illustrative.
3. TX and RX are separate. The implemented numerical screen geometry requires an x-parallel baseline and a crossing strictly between TX and RX.
4. The comparison engine and app use the shared channel. The older standalone demo retains its own channel code.
5. Reflected paths are optional and disabled by default. Their parameters are assumptions; the channel has no site-derived reflection geometry.
6. Calibration uses target-free synthetic recordings, with independent evaluation seeds under the supplied default protocol. This does not establish performance on an unseen physical site.
7. No measured RF dataset, SDR acquisition integration or archived MATLAB result/log package is present in the reviewed source.
8. The upstream text explicitly labels its reference numbers as NumPy-transcription predictions and says MATLAB execution is pending. Inherited comments saying “verified” do not establish a new execution record.
9. The upstream text's seven-file instruction describes the historical physics subset. The full repository has 12 MATLAB files.
10. The Fresnel source comment references an absent `README_MATLAB.txt`. Attribution in the source and the supplied `README_MATLAB_v2.txt` were retained.

## Changes and verification

The update replaces README with a complete project description, evidence boundaries, team feasibility assessment, hardware/measurement roadmap, full file inventory and runnable entry points. This review record is added. All 12 MATLAB files and the upstream reference text were checked against their source Git blob hashes and remain byte-for-byte unchanged.

MATLAB and Octave were unavailable locally during this update. No physics, channel, detector or app test is reported as run or passed. README commands were checked against the source entry points, option names, configuration fields and saved-result variable names.

The MATLAB checks remain the next reproduction milestone. Run them individually, archive their complete summaries with the MATLAB release and commit, and investigate failures before publishing numerical performance claims. A successful mathematical check still does not replace RF measurement validation.

Field feasibility is the team's provisional engineering judgment. Demonstrating repeatable known-obstruction signatures is the first experimental gate; drone detection, classification and deployment reliability require later independent evidence.
