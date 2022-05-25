# unity-yaml-rust

`unity-yaml-rust` is a pure Rust YAML parser for Unity.

Add Unity YAML spec and mutable access Base on [yaml-rust](https://github.com/chyh1990/yaml-rust) 


## Quick Start

Add the following to the Cargo.toml of your project:

```toml
[dependencies]
unity-yaml-rust = "0.1"
```

Use `yaml::YamlLoader` to load the YAML documents and access it
as Vec/HashMap:

```rust
use unity_yaml_rust::{yaml::YamlLoader, emmitter::YamlEmitter};

fn main() {
    let s =
r#"
%YAML 1.1
%TAG !u! tag:unity3d.com,2011:
--- !u!687078895 &4343727234628468602
SpriteAtlas:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_Name: atlas_launch
  m_EditorData:
    serializedVersion: 2
    textureSettings:
      serializedVersion: 2
      anisoLevel: 1
      compressionQuality: 50
      maxTextureSize: 2048
      textureCompression: 0
      filterMode: 1
      generateMipMaps: 0
      readable: 0
      crunchedCompression: 0
      sRGB: 1
    platformSettings: []
    packingSettings:
      serializedVersion: 2
      padding: 4
      blockOffset: 1
      allowAlphaSplitting: 0
      enableRotation: 0
      enableTightPacking: 0
    variantMultiplier: 1
    packables:
    - {fileID: 102900000, guid: c5a32d8209c314631bad292a32582dfc, type: 3}
    bindAsDefault: 1
  m_MasterAtlas: {fileID: 0}
  m_PackedSprites:
  - {fileID: 21300000, guid: 3083aff0bd42a4268a9cfe6e564ab247, type: 3}
  - {fileID: 21300000, guid: 054656e6c52c2425eb7c5ec764d03587, type: 3}
  - {fileID: 21300000, guid: 55aba929877c26747acf9ad10ee7989c, type: 3}
  m_PackedSpriteNamesToIndex:
  - login_ic_logo
  - launch_icon_service
  - login_ic_logo_bak1
  m_Tag: atlas_launch
  m_IsVariant: 0
"#;
    let mut docs = YamlLoader::load_from_str(s).unwrap();

    // Multi document support, doc is a yaml::Yaml
    for doc in docs.iter_mut() {
        // Debug support
        println!("{:?}", doc);

        if !matches!(doc, Yaml::Original(_)) {
            //IndexMut
            let sprite_atlas = &mut doc["SpriteAtlas"];
            
            assert_eq!(sprite_atlas["m_ObjectHideFlags"].as_i64(), Some(0i64));
            assert!(sprite_atlas["m_ObjectHideFlags"].replace_i64(3));
            assert_eq!(sprite_atlas["m_ObjectHideFlags"].as_i64(), Some(3i64));
            
            sprite_atlas["m_Name"].replace_string("launch".to_string());
            assert_eq!(sprite_atlas["m_Name"].as_str(), Some("launch"));

            sprite_atlas.insert("custom", Yaml::Boolean(true));
            assert_eq!(sprite_atlas["custom"].as_bool(), Some(true));

            sprite_atlas.remove("m_PackedSprites");
            assert!(sprite_atlas["m_PackedSprites"].is_badvalue());

            sprite_atlas["m_EditorData"]["packables"].remove_at(0);
            sprite_atlas["m_EditorData"]["packables"].push(Yaml::String("ppp".to_string()));
            sprite_atlas["m_MasterAtlas"].insert("plus", Yaml::Boolean(false));
            sprite_atlas["m_MasterAtlas"].remove("fileID");
        }

        // Dump the YAML object
        let mut out_str = String::new();
        {
            let mut emitter = YamlEmitter::new(&mut out_str);
            emitter.dump(doc).unwrap(); // dump the YAML object to a String
        }
        println!("{}", out_str);
    }
}
```

## License

Licensed under either of

 * Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Fork & PR on Github.

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any
additional terms or conditions.
