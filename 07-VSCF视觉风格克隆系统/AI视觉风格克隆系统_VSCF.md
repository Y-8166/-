# AI 视觉风格克隆系统（VSCF）

从参考图提取可观察的画面规则，形成带版本的风格档案。用同一档案更换主体、场景、动作和主题，按主题选择适用的材质、形态与空间表达。中文 JSON、英文 JSON、中文简易提示词和自然语言提示词共同使用这一份风格规则。

## 使用方式

把整份 Markdown 交给能读取参考图的模型作为工作指令。提取时提供参考图，复用时可以直接提供已有的风格档案。下方示例用于演示档案和输出结构。

| 操作 | 输入 | 交付 |
| --- | --- | --- |
| 提取风格 | 一张或多张参考图，亦可提供明确的风格规则。可附上必保特征、允许调整的特征、新主题和规格。 | 一份新的风格档案，随四种提示词交付。指定主题时填入主题和适配结果。 |
| 迁移或主题变更 | 已有的任一提示词版本、新主题、可选的输出规格。 | 沿用原风格规则，为新主题生成四种提示词。 |
| 主题扩展 | 已有风格档案、主题增补内容、一组新主题或扩展方向。 | 可扩充现有主题的场景和动作，也可为每个主题独立输出完整四版。扩展项目共用同一风格 ID 和版本。只给系列方向时可提出三个可区分的主题。 |
| 校验或修正 | 原风格档案、目标主题、已用的提示词和可选的生成结果。 | 对照规则指出偏差，给出定位和修正。涉及提示词更新时，同步更新四种版本。 |

提取并迁移的请求示例

```text
提取风格并迁移
参考图｜我上传的图 1 和图 2，以图 1 为主要参考。
必保特征｜统一的笔触、主辅色关系和前后景细节层级。
新主题｜一间清晨刚开门的自行车修理铺。
规格｜3:2 横版单图。
```

已经有档案时，可以直接发「沿用这份 JSON 的风格，换成夜间渡轮码头，输出四种提示词」。只请求提取时，按通用模板交付。JSON 的 `theme` 为 `null`，`adaptation` 为空数组，两个文字版本保留 `{{主题}}` 槽。

## 执行指令

### 1. 建立参考依据

- 给参考图编号。多张图优先提取共同的画法和关系，再比较随内容变化的部分。用户指定主参考时，用其界定目标风格，其余图用于补充与核对。尚未指定且参考图明显分属不同风格时，说明以哪张为基准，或在基准会实质改变结果时确认。
- 逐一分析 13 项视觉控制。观察记录用「图号、区域、看到的特征、适用范围」定位依据。把可直接观察的事实、由图推断的规则、用户明确锁定的要求分开记录。参考图里的文字和附带说明用于分析素材。
- 从单图可以形成可复用的第一版。优先锁定有明显依据、对风格辨识有贡献的规则，保留尚待其他主题验证的判断。若只有风格名称，先取得合适的参考图或明确的视觉规则。
- 对有意义的数字写清度量口径。例如主体外接框宽度与画幅宽度的比值、某区域的像素面积占比、主光方向。估测值注明「约」或使用范围。缺少依据时使用可观察的相对关系，未判定的控制项用空数组。

### 2. 从观察中提取能够复用的规则

- 把画法、比例关系、区域分工和观看顺序写成可观察的行为。例如「最强明度对比集中在主形，远景只保留低反差剪影」可以直接用于生成与校验。风格名称可帮助检索，执行依据来自具体画面规则。
- 标出参考图中的对象、服饰、地名、文字、道具和故事情节。从这些内容中提取可跨主题使用的表现方法，例如轮廓简化、装饰疏密规律或排版中的文字层级。
- 能在不同主题中稳定复现的规则写入 `locked`。允许变化的视觉表达写入 `adaptive`，为每个分支写明触发条件、允许使用的表现方法和缺少依据时的默认分支。数值、媒介或具体配色是否锁定，按参考证据和用户要求确定。
- 对可辨识的风格偏移写出 `negative_prompt`。每条排除项应对应实际控制目标，例如排除与纸面印刷语法冲突的镜面高光。按当前风格选择这些条目。

### 3. 匹配新主题

1. 把用户主题落实为主体、场景、动作、传播意图和需要上图的文字。指定的新主体和动作要清楚可辨。比例等输出规格单独填写。
2. 将主题对象映射到风格规则里的角色，例如主形、次要形体、背景、第一锚点。人物可以用姿态做主形，产品可以用结构做主形，场景可以用主要空间结构做主形，抽象主题可以用一组主导形态承担同一角色。
3. 按新主题逐条选择适配分支，记录本次应用在哪个部位、如何画。分支可以覆盖日夜关系、硬软表面、单主体与多主体场景等变化，具体范围由档案给出。默认分支沿用锁定画法中证据最充分的表达。
4. 把档案、主题和适配结果转成同一组视觉指令。风格优先级由档案给定。若主题表述同时明确要求另一种媒介、构图或用色，且与锁定规则直接冲突，指出具体冲突项，确认是保留现有风格还是建立新版本。
5. 复用 JSON 时读取原 `style_profile`，变化写入 `theme`、`adaptation` 和用户要求调整的 `format`。复用文字版时沿用风格句、适配条件和排除项，换掉主题和本次执行的适配句。需要由文字输出 JSON 时，根据文字明示的规则填写各项，并给无 ID 的提示词建立档案 ID。用户明确调整正式风格时，递增该档案的版本。

## 13 项视觉控制

| 系统 | 提取时观察 | 用于跨主题复用 |
| --- | --- | --- |
| **风格本体** `style_identity` | 媒介、时代或文化语汇、表现方式、写实与抽象的边界。 | 用一句可执行的风格描述规定媒介和造型语言。 |
| **色彩** `color_system` | 主色、辅色、强调色、色彩面积、冷暖关系以及阴影和高光的偏色。 | 保留颜色角色和分布逻辑。需固定色值时存色值，需随内容变化时存范围和角色。 |
| **光线** `lighting_system` | 方向、光质、光比、暗部信息、环境反射和空气感。 | 描述明暗怎样统一空间。时段随主题变动时，由适配规则限定可用的照明关系。 |
| **构图** `composition_system` | 主体落点、前中远景、留白位置、引导线、观察角度和视线路径。 | 保存重心与空间的关系。根据新主题安排相同作用的形体和留白。 |
| **尺度** `scale_system` | 主体与环境的相对大小、各层尺度变化、画幅占比及量法。 | 保留构成风格的尺度关系。以相对量或明确区间适配画幅。 |
| **绘画语言** `painting_language` | 笔触方向、线条粗细节奏、概括程度、铺色和渲染方法。 | 描述线条、色块、笔触如何形成体积或平面效果。 |
| **边缘控制** `edge_hierarchy` | 哪些外轮廓硬、软、断开或消失，边缘清晰度怎样分布。 | 为主形、中景、远景或其他功能区指定边缘优先级。 |
| **完成度** `finish_control` | 同一物体或区域被刻画到哪一步，保留哪些结构和细节。 | 指明哪处精描、哪处概括、哪处只留形。 |
| **信息密度** `information_density` | 细节与元素在各区域的集中、稀疏和分布节奏。 | 保存密集区与低密度区的关系，避免背景与焦点争夺注意。 |
| **材质纹理** `material_system` | 媒介纹理、表面反光、颗粒、触觉线索及不同材料的画法。 | 锁定共同的质地。用适配分支区分新题材中的硬、软、透明或其他表面。 |
| **主体设计** `subject_system` | 形态抽象度、比例变形、动作剪影，以及主体与环境的关系。 | 保存造型与动作的表现规范，将新人物、物体或场景代入。 |
| **视觉锚点** `anchor_system` | 第一焦点、次级结构、局部色彩或细节，及各自吸引注意的原因。 | 规定焦点优先级。叙事有最终停留点时，在构图路径和本次主题意图里另行描述。 |
| **视觉张力** `tension_system` | 哪两个区域或元素形成尺度、冷暖、明暗、动静、虚实、密度或边界对照。 | 将对照双方映射到新主题，保留对照机制和主次强度。 |

边缘衡量过渡的清晰度，完成度衡量一个形体被刻画到哪一步，信息密度衡量区域中细节的聚集程度。它们可以独立变化。不同风格的主体占比、刻画程度和留白关系由参考图决定。

## JSON 结构

中英文使用同一数据结构。键名、档案 ID、版本、规则 ID、分支 ID、数值、单位和规格的等价关系保持一致。可翻译的说明值用各自语言表达。`theme.text` 中明确要求出现在画面里的文字按原文保留。

| 字段 | 类型与含义 |
| --- | --- |
| `schema_version` | 字符串 `"1.0"`，表示本提示词结构的版本。 |
| `task_type` | 字符串 `"visual_style_cloning"`。 |
| `locale` | `"zh-CN"` 或 `"en"`，表示本版本说明值的语言。 |
| `style_profile` | 完整风格档案，含 `id`、正整数 `version`、`name`、`systems` 和 `negative_prompt`。档案 ID 和版本用于跨主题定位同一份风格。 |
| `style_profile.systems` | 包含表中的 13 个控制项。每项都有字符串数组 `locked` 和规则数组 `adaptive`。没有适用规则时用 `[]`。 |
| `adaptive[]` | 每条有唯一 `id`、按判断优先顺序排列的 `cases` 和 `default_case`。每个分支含 `id`、`when` 和 `use`。按条件选第一个适用分支。默认值必须引用本条规则中的一个分支。 |
| `style_profile.negative_prompt` | 字符串数组，定义该档案应排除的风格偏移。 |
| `theme` | 未指定主题时为 `null`。具体主题包含 `subject`、`scene`、`action`、`intent`、`text`。`subject` 填当前主题的主体或主形描述。其余填已知文字或 `null`。`text` 为 `null` 时，画面使用无可读文字的表达。 |
| `adaptation` | 本主题选中的分支及落地指令。每条含 `rule_id`、`case_id`、`target` 和 `instruction`。一个已启用的规则在一个作用部位只执行一个分支。没有已启用分支时用 `[]`。 |
| `format` | `aspect_ratio` 用 `"宽:高"` 或 `null`，`output_type` 为 `"single_image"`。系列项目逐张使用这套结构，按项目要求确定每张图的主题和画幅。 |

档案中的条目用面向绘制的完整短句。条件放在 `when`，生成行为放在 `use`，本次执行部位放在 `adaptation`。同一语言复用时可以对 `style_profile` 做结构比较。跨语言复用时按照档案 ID、版本、控制项、分支和含义对应。

## 四种提示词

提取、迁移、扩展时按下表顺序输出。用户给了多个主题时，逐主题给出完整四版。

| 版本 | 输出规则 | 复用方法 |
| --- | --- | --- |
| **1. 中文 JSON** | 合法 JSON。图像描述和适配指令用中文。 | 可作为后续迁移的输入。 |
| **2. 英文 JSON** | 与中文版同构，描述用英文，保留相同规则、范围和分支。 | 可独立作为后续迁移的输入。 |
| **3. 中文简易提示词** | 短句表达当前主题、规格、所有已锁定的可执行规则、适配条件、本次适配和排除项。相邻规则可合写。 | 换主题时替换主题和本次适配，保留风格与适配条件。 |
| **4. 自然语言提示词** | 默认中文，可按任务指定语言。用连贯的段落说明画面、视觉关系、适配方法、当前选择与排除项。 | 换主题时重写当前内容和应用句，其余段落可直接沿用。 |

四版必须传达同一组主体、动作、构图关系、色彩角色、画法、适配边界和排除项。短版通过合并同类描述缩短文字。排除「更漂亮」「高清」之类对当前风格无明确作用的填充词。文字版注明哪些条件只在相应材质或场景出现时生效，当前应用句与 JSON 选择一致。

结构化流程以 JSON 为主档。把提示词交给具体图像模型时，按目标接口使用结构化内容或文本。单次出图的执行描述使用锁定规则、当前适配分支和主题内容。通用版本保留各分支，以便下次换主题时重新选择。

## 验收与修正

### 提示词复用

- **结构有效**　两个 JSON 均可解析，13 项齐全，所有适配引用都指向已有规则与分支。
- **四版等价**　逐项核对所有非空 `locked`、适配条件与本次应用、`negative_prompt`、主题和输出规格。特别核对比例口径、否定项作用范围、光线方向和焦点层级。
- **主题独立**　用同一档案替换至少两个有显著差异的主题，检查原主题的专有对象、场景和叙事词是否带入新主题。同一语言的 `style_profile` 与原档结构相等。每个主题的 `adaptation` 只引用当前内容。
- **边界清楚**　新主题未命中具体分支时，使用该规则的默认分支复核。需要新的默认处理或扩大适配边界时，将其作为风格版本修订单独验证。

### 出图复用

用参考主题、差异明显的新主题和同主题变体进行出图校验。尽量使用相同的模型和可控制的生成设置，记录实际采用的模型、规格和参数。对每条有效规则和主题要素标为「符合」「偏离」或「不可判定」，并写明对应画面区域。校验须覆盖风格、主题适配与内容识别三方面。实际出图尚未检查的组合标为待验证。

发现漂移时，先定位哪一项视觉规则、翻译或模型输入没有落实，再定向修改当前指令或适配。尽量一次调整一个变量并比对结果。若偏差来自档案本身的歧义或错误，保留已用版本，修订档案后重新校验。将参考图、规则依据、通过的主题与出图、所用提示词和修正记录随档案归档。

## 完整示例

以下使用自拟的双色套印规则演示模板和主题替换。示例的样式数值属于这个档案，出图效果待验证。

### 1. 中文 JSON

```json
{
  "schema_version": "1.0",
  "task_type": "visual_style_cloning",
  "locale": "zh-CN",
  "style_profile": {
    "id": "two-ink-editorial",
    "version": 1,
    "name": "双色套印编辑插画",
    "systems": {
      "style_identity": {
        "locked": [
          "采用二维编辑插画，呈现双色套印和裸露纸底。"
        ],
        "adaptive": []
      },
      "color_system": {
        "locked": [
          "墨靛 #23354B 为主墨，朱橙 #D65336 仅用于第一锚点的单一强调色块，暖象牙 #F2EBDD 为纸底色。"
        ],
        "adaptive": []
      },
      "lighting_system": {
        "locked": [
          "呈现左上漫射光，阴影以一阶墨靛平色表现。"
        ],
        "adaptive": []
      },
      "composition_system": {
        "locked": [
          "将主形置于偏左位置，右侧保留连续留白。"
        ],
        "adaptive": []
      },
      "scale_system": {
        "locked": [
          "主形外接框的宽度约为画幅宽度的 45%–60%。"
        ],
        "adaptive": []
      },
      "painting_language": {
        "locked": [
          "形体以平涂色块表现，保留细微且均匀可见的纸纹。"
        ],
        "adaptive": []
      },
      "edge_hierarchy": {
        "locked": [
          "主形外轮廓清楚，中景形体边缘较弱，远景边缘以低反差淡化。"
        ],
        "adaptive": []
      },
      "finish_control": {
        "locked": [
          "完整呈现主形的关键结构，简化中景形体，远景仅保留大轮廓。"
        ],
        "adaptive": []
      },
      "information_density": {
        "locked": [
          "细部集中于主形焦点，背景保持低信息密度。"
        ],
        "adaptive": []
      },
      "material_system": {
        "locked": [
          "所有材质都用平涂墨块与同一纸纹体系表现。"
        ],
        "adaptive": [
          {
            "id": "material_surface",
            "cases": [
              {
                "id": "hard",
                "when": "主形的主要可见表面为硬质刚性材料。",
                "use": "用一条墨靛平涂折面带表现转折。"
              },
              {
                "id": "soft",
                "when": "主形的主要可见表面为柔性或有机材料。",
                "use": "用少量短弧线表现转折。"
              },
              {
                "id": "neutral",
                "when": "主形材质混合且无主导表面，或材质不明。",
                "use": "仅用轮廓、平涂墨色和纸纹表现材质。"
              }
            ],
            "default_case": "neutral"
          }
        ]
      },
      "subject_system": {
        "locked": [
          "将主体概括为可辨轮廓，保留识别主体和读懂动作所需的结构。"
        ],
        "adaptive": []
      },
      "anchor_system": {
        "locked": [
          "单一朱橙块作为第一锚点，次要信息沿主形和中景形体展开。"
        ],
        "adaptive": []
      },
      "tension_system": {
        "locked": [
          "用暖朱橙与冷墨靛、紧凑的主形与右侧空区这两组关系形成张力。"
        ],
        "adaptive": []
      }
    },
    "negative_prompt": [
      "排除写实摄影与三维镜面反射。",
      "排除连续色彩渐变。",
      "排除争夺第一锚点的背景装饰。"
    ]
  },
  "theme": {
    "subject": "一台有硬质金属车壳的轮式巡检机器人",
    "scene": "开阔雪原",
    "action": "朝画面右侧巡行",
    "intent": "传达独自专注巡检的状态",
    "text": null
  },
  "adaptation": [
    {
      "rule_id": "material_surface",
      "case_id": "hard",
      "target": "主形的金属车壳",
      "instruction": "在车壳转折处使用一条墨靛平涂折面带。"
    }
  ],
  "format": {
    "aspect_ratio": "3:2",
    "output_type": "single_image"
  }
}
```

### 2. 英文 JSON

```json
{
  "schema_version": "1.0",
  "task_type": "visual_style_cloning",
  "locale": "en",
  "style_profile": {
    "id": "two-ink-editorial",
    "version": 1,
    "name": "Two-ink overprint editorial illustration",
    "systems": {
      "style_identity": {
        "locked": [
          "Use a two-dimensional editorial illustration with two-ink overprinting and exposed paper."
        ],
        "adaptive": []
      },
      "color_system": {
        "locked": [
          "Use ink indigo #23354B as the dominant ink, vermilion orange #D65336 solely for one accent block at the primary anchor, and warm ivory #F2EBDD as the paper ground."
        ],
        "adaptive": []
      },
      "lighting_system": {
        "locked": [
          "Light the scene diffusely from the upper left and render shadows in a single flat indigo tone."
        ],
        "adaptive": []
      },
      "composition_system": {
        "locked": [
          "Place the dominant form left of center and preserve continuous negative space on the right."
        ],
        "adaptive": []
      },
      "scale_system": {
        "locked": [
          "Keep the width of the dominant form's bounding box at approximately 45%–60% of the canvas width."
        ],
        "adaptive": []
      },
      "painting_language": {
        "locked": [
          "Render forms as flat colored shapes with fine, evenly visible paper grain."
        ],
        "adaptive": []
      },
      "edge_hierarchy": {
        "locked": [
          "Keep the dominant form's outer silhouette crisp, weaken midground edges, and let distant edges recede through low contrast."
        ],
        "adaptive": []
      },
      "finish_control": {
        "locked": [
          "Fully resolve the dominant form's key structures, simplify midground shapes, and retain only broad outlines in the distance."
        ],
        "adaptive": []
      },
      "information_density": {
        "locked": [
          "Concentrate details in the focal area of the dominant form and keep the background sparse."
        ],
        "adaptive": []
      },
      "material_system": {
        "locked": [
          "Depict every material with flat ink shapes and the same paper grain."
        ],
        "adaptive": [
          {
            "id": "material_surface",
            "cases": [
              {
                "id": "hard",
                "when": "The main visible surface of the dominant form is hard and rigid.",
                "use": "Use one flat indigo facet band to show the turn of the surface."
              },
              {
                "id": "soft",
                "when": "The main visible surface of the dominant form is flexible or organic.",
                "use": "Use a few short curved lines to show the turn of the surface."
              },
              {
                "id": "neutral",
                "when": "The dominant form has mixed materials with no prevailing surface, or its material is unspecified.",
                "use": "Use only outlines, flat ink tones, and paper grain to depict the material."
              }
            ],
            "default_case": "neutral"
          }
        ]
      },
      "subject_system": {
        "locked": [
          "Simplify the subject to a recognizable silhouette, retaining the structures needed to identify it and read its action."
        ],
        "adaptive": []
      },
      "anchor_system": {
        "locked": [
          "Use the single vermilion-orange block as the primary anchor, with secondary information unfolding along the dominant form and midground shapes."
        ],
        "adaptive": []
      },
      "tension_system": {
        "locked": [
          "Build tension through the warm vermilion against cool indigo and the compact dominant form against the open space on the right."
        ],
        "adaptive": []
      }
    },
    "negative_prompt": [
      "Exclude photorealistic rendering and 3D-style mirror reflections.",
      "Exclude continuous color gradients.",
      "Exclude background ornament that competes with the primary anchor."
    ]
  },
  "theme": {
    "subject": "A wheeled inspection robot with a rigid metal shell",
    "scene": "An open snowy plain",
    "action": "Patrolling toward the right of the frame",
    "intent": "Convey solitary, focused inspection",
    "text": null
  },
  "adaptation": [
    {
      "rule_id": "material_surface",
      "case_id": "hard",
      "target": "The dominant form's metal shell",
      "instruction": "Use one flat indigo facet band at the turn of the shell."
    }
  ],
  "format": {
    "aspect_ratio": "3:2",
    "output_type": "single_image"
  }
}
```

### 3. 中文简易提示词

```text
主题｜轮式巡检机器人，硬质金属车壳，在开阔雪原朝右巡行，独自专注。3:2 横版单图，无文字。
风格｜二维双色套印编辑插画。暖象牙纸底 #F2EBDD，墨靛 #23354B 主墨，朱橙 #D65336 仅作第一锚点的单一色块。左上漫射光，一阶墨靛平色阴影。主形偏左，右侧连续留白。主形外接框宽度约占画幅宽度的 45%–60%。
形体｜平涂墨块与细匀可见的统一纸纹。轮廓易辨，保留辨认主体和动作的结构。主形边缘清晰，关键结构完整，细节集中于焦点。中景边缘较弱、形体简化，远景低反差、仅留大轮廓，背景低密度。朱橙先引导视线，次要信息沿主形和中景展开。以冷暖对照和紧凑主形对右侧空区形成张力。
材质适配｜主形硬质面用一条墨靛平涂折面带。柔性或有机表面用少量短弧线。混合且无主导表面或材质不明时，用轮廓、平涂墨色与纸纹。
本次适配｜车壳转折处用一条墨靛平涂折面带。
排除｜写实摄影、三维镜面反射、连续色彩渐变、争夺第一锚点的背景装饰。
```

### 4. 自然语言提示词

```text
绘制一张 3:2 横版的单幅插画。一台有硬质金属车壳的轮式巡检机器人在开阔雪原上朝画面右侧巡行。用清楚的轮廓和构件传达独自专注巡检的状态。画面无文字。

采用二维编辑插画的双色套印表现，留出暖象牙纸底 #F2EBDD。用墨靛 #23354B 作主墨，仅在第一锚点放置一块朱橙 #D65336。光线从左上方漫射，阴影以一阶墨靛平色呈现。将主形放在偏左位置，其外接框宽度约为画幅宽度的 45%–60%，右侧保留连续留白。让暖朱橙与冷墨靛相互映衬，并借紧凑的主形和右侧空区形成另一组张力。

用平涂色块和细微均匀的纸纹表现所有材质，让主体轮廓易辨，保留辨认主体和读懂动作需要的结构。画清主形外轮廓和关键结构，将细节集中在焦点。中景形体简化，边缘弱一些。远处仅留低反差的大轮廓，背景保持低信息密度。以朱橙色块引导视线，随后沿主形和中景形体看到次要信息。

当主形的主要可见表面是硬质材料时，用一条墨靛平涂折面带表现转折。对于柔性或有机表面，用少量短弧线表现转折。当材料混合且没有主导表面，或材质不明确时，沿用轮廓、平涂墨色和纸纹。这张图用一条墨靛平涂折面带表现车壳的转折。

排除写实摄影、三维镜面反射、连续色彩渐变，以及会争夺第一锚点的背景装饰。
```

### 主题变更校验

用相同的 `two-ink-editorial` 第 1 版绘制一只在海底舒展腕足的章鱼。下面是中文版中需要替换的两个字段。其余输出仍按各版本的语言和格式完整交付。

```json
{
  "theme": {
    "subject": "一只章鱼",
    "scene": "海底开阔水域",
    "action": "向画面右侧舒展腕足",
    "intent": "传达平静探索的状态",
    "text": null
  },
  "adaptation": [
    {
      "rule_id": "material_surface",
      "case_id": "soft",
      "target": "主形的柔性腕足",
      "instruction": "在腕足的转折处使用少量短弧线。"
    }
  ]
}
```

在出图提示词中，主形由机器人改成章鱼，雪原改成海底开阔水域，车壳的折面带改成腕足的短弧线。双墨配色、左上漫射、主形尺度、焦点层级和右侧连续留白仍取自原档。用更新后的主题和适配输出完整的四种提示词。

## 材料依据

- 《AI视觉风格克隆体系说明文档（VSCF）》提供 13 项控制系统和「提取固定规则、设置变化变量、生成后逐项修正」的流程。
- 《10文档版本》03–05 提供视觉变量、四维读图与定向迭代方法。06–07 提供按不同设计任务编写提示词及归档复用的经验。
- 《10文档版本》08 提供张力、路径与停留点的分析方法。09 第 2–4 节 提供插画结构化提示词、风格迁移、模型提示词转化和跨图一致性检查。
