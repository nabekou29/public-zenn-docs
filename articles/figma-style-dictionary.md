---
title: "Style Dictionary で Figma のデザイントークンを管理する"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - 'Figma'
  - 'frontend'
  - 'styledictionary'
  - 'design'
  - 'デザイントークン'
  - 'デザインシステム'
published: true
---

:::message
[Unipos Advent Calender 2024](https://qiita.com/advent-calendar/2024/unipos) 15日目の記事です
:::

今回は、Style Dictionary を使って、JSON や YAML で管理されたデザイントークンを Figma にインポートする方法を紹介します。

使用したコードは以下のリポジトリにあります。

https://github.com/nabekou29/figma-style-dictionary-sample

# ゴール

- デザイントークンを YAML で管理する
- Figma にインポートできるようにする
  - 基本は Variables としてインポートする
  - Variables が対応していないものは、Styles としてインポートする
  - トークンが他のトークンを参照している場合は、Figma でも参照が維持されるようにする


以下のように、Figma に読み込まれる状態を目指します

| Variables | Styles |
| --- | --- |
| ![Variables](/images/figma-style-dictionary/figma-variables.png)  | ![Styles](/images/figma-style-dictionary/figma-styles.png) |

# デザイントークンとは

[Design Tokens Community Group](https://github.com/design-tokens/community-group?tab=readme-ov-file#design-tokens) によると、次のように説明されています。

> Design tokens are indivisible pieces of a design system such as colors, spacing, typography scale.
> (訳) デザイントークンとは、色、余白、タイポグラフィスケールなど、デザインシステムを構成する分割不可能な要素のことです。

どのようなトークンを定義するかは、プロジェクトによって異なりますが、今回はカラーパレットや使うフォントの１つ１つが、１つのトークンとして定義されているという理解があれば大丈夫です。

イメージは以下のような感じです。

| トークン名 | 値 |
| --- | --- |
| color-red | #ff0000 |
| color-blue | #0000ff |
| font-size-sm | 12px |
| font-size-md | 16px |

# Style Dictionary とは

[Style Dictionary](https://styledictionary.com/) は、デザイントークンを管理して、複数のプラットフォームにエクスポートするためのツールです。
JSON や YAML から css や js, Swift などの形式に変換することができます。

すでにご存知の方は読み飛ばしてください。

## 試しに Style Dictionary を使ってみる

Style Dictionary のイメージを掴むために、トークンを定義して css 向けに変換してみようと思います。

まずは以下のようにトークンを定義します。

フォーマットは Design Tokens Community Group (以下、DTCG) が提案している仕様に合わせます。
*Style Dcitionary が一部 DTCG の仕様に対応していない箇所がありそうなので注意が必要です。*
( `$type: dimension` の `$value` の書き方など）

```yaml:tokens.yaml
color:
  $type: color
  white:
    $value: "#ffffff"
  black:
    $value: "#000000"
  red:
    200:
      $value: "#ffc9c9"
    400:
      $value: "#fb6e6e"
    600:
      $value: "#e02222"
    800:
      $value: "#9c1818"
  blue:
    200:
      $value: "#c5daf8"
    400:
      $value: "#70a2ea"
    600:
      $value: "#3964d7"
    800:
      $value: "#2d43a0"

font-size:
  $type: dimension
  sm:
    $value: 14
  md:
    $value: 16
  lg:
    $value: 20

font-weight:
  $type: fontWeight
  normal:
    $value: 400
  bold:
    $value: 700

font-family:
  $type: fontFamily
  helvetica:
    $value: ["Hiragino Sans", "sans-serif"]

typography:
  $type: typography
  body:
    $value:
      fontFamily: "{font-family.helvetica}"
      fontSize: "{font-size.md}"
      fontWeight: "{font-weight.normal}"
  heading:
    $value:
      fontFamily: "{font-family.helvetica}"
      fontSize: "{font-size.lg}"
      fontWeight: "{font-weight.bold}"
```

次にビルド用の簡単な実装をします。

```typescript:build.ts
import yaml from 'yaml';

import StyleDictionary from 'style-dictionary';
import type { Parser } from 'style-dictionary/types';

// --- Register Parser ---

const yamlParser: Parser = {
  name: 'yaml-parser',
  pattern: /\.yaml$/,
  parser: ({ contents }) => yaml.parse(contents),
};
StyleDictionary.registerParser(yamlParser);

// --- Config ---

const sd = new StyleDictionary({
  parsers: ['yaml-parser'],
  source: ['tokens.yaml'],
  platforms: {
    css: {
      transformGroup: 'css',
      transforms: ['size/px'],
      files: [
        {
          destination: 'build/variables.css',
          format: 'css/variables',
          options: { outputReferences: true },
        },
      ],
    },
  },
});

// --- Build ---

await sd.hasInitialized;

await sd.cleanAllPlatforms();
await sd.buildAllPlatforms();
```

`package.json` などを含めた全体のコードは以下を参照してください。

https://github.com/nabekou29/figma-style-dictionary-sample/tree/css_variables/design-token

### 結果

このように、トークンが css の変数として出力されます。

```css:build/variables.css
/**
 * Do not edit directly, this file was auto-generated.
 */

:root {
  --color-white: #ffffff;
  --color-black: #000000;
  --color-red-200: #ffc9c9;
  --color-red-400: #fb6e6e;
  --color-red-600: #e02222;
  --color-red-800: #9c1818;
  --color-blue-200: #c5daf8;
  --color-blue-400: #70a2ea;
  --color-blue-600: #3964d7;
  --color-blue-800: #2d43a0;
  --font-size-sm: 14px;
  --font-size-md: 16px;
  --font-size-lg: 20px;
  --font-weight-normal: 400;
  --font-weight-bold: 700;
  --font-family-helvetica: 'Hiragino Sans', sans-serif;
  --typography-body: var(--font-weight-normal) var(--font-size-md) var(--font-family-helvetica);
  --typography-heading: var(--font-weight-bold) var(--font-size-lg) var(--font-family-helvetica);
}
```

# Figma にインポートする

Figma のプラグインのサンプルとして、デザイントークンをインポートするプラグインが公開されています。

https://github.com/figma/plugin-samples/blob/master/variables-import-export/README.md

ただし、もとの状態では**色と数値を Variables としてインポートすることしかできません。**
そのため、プラグインを自前で作成する必要があります。

## 方針

YAML の定義を Figma にインポートするにあたり、**２つの方針が考えられます。**

1. **プラグインで、DTCG の形式を解釈できるようにする**
2. **Style Dictionary でプラグインが解釈しやすい形式に変換する**

1 はもとのサンプルのプラグインに近い形式ですが、今回は 2 を採用します。

理由は以下の通りです。

- Style Dictionary のユーティリティを活用すると楽に変換できる
- 他プラットフォーム向けの変換のカスタマイズを Figma にも適用しやすい
- プラグインの処理を複雑にするなら TypeScript で書きたいが、いちいちビルドするのが面倒

## 実装

### Style Dictionary の設定

Style Dictionary の設定を変更して、Figma にインポートしやすい形式に変換します。

細かく説明しようとすると長くなるので今回は省略します。（別途記事にするかもしれません）
基本的には Figma 向けの変換やフォーマットを自作しているだけです。

いくつかの工夫によってサンプルのプラグインで行っていた複雑な処理を簡略化しています。

- 参照をうまく解決するために、`style-dictionary/utils` の関数を使う
- 色を Figma 向けに変換するために、`color2k` を使う

また、Variables に関しては Scope を指定することで、Figma で使用できる箇所を限定するようにしています。

:::details build.ts

```typescript:build.ts
import yaml from 'yaml';

import StyleDictionary from 'style-dictionary';
import type {
  Dictionary,
  Format,
  OutputReferences,
  Parser,
  TransformedToken,
  ValueTransform,
} from 'style-dictionary/types';
import { sortByReference, usesReferences } from 'style-dictionary/utils';

import type {
  Effect,
  StyleType,
  VariableResolvedDataType,
  VariableScope,
} from '@figma/plugin-typings/plugin-api-standalone.js';
import { parseToRgba } from 'color2k';

// --- Register Parser ---

const yamlParser: Parser = {
  name: 'yaml-parser',
  pattern: /\.yaml$/,
  parser: ({ contents }) => yaml.parse(contents),
};
StyleDictionary.registerParser(yamlParser);

// --- For Figma ---

type FigmaTokenBase = {
  name: string;
  $value: unknown;
  $description?: string;
};

/** Figma向けのトークン */
export type FigmaToken = FigmaTokenBase &
  (
    | {
        tokenType: 'variable';
        type: VariableResolvedDataType;
        name: string;
        scopes?: VariableScope[];
      }
    | {
        tokenType: 'style';
        type: StyleType;
        effectType?: Effect['type'];
      }
  );

/** 参考: https://github.com/amzn/style-dictionary/blob/v4.2.0/lib/common/formatHelpers/formattedVariables.js */
export const formattedFigmaToken = ({
  dictionary,
  outputReferences,
  usesDtcg = false,
}: {
  dictionary: Dictionary;
  outputReferences?: OutputReferences;
  usesDtcg?: boolean;
}) => {
  const tokens = dictionary.tokens;

  const allTokens = outputReferences
    ? dictionary.allTokens.toSorted(
        sortByReference(tokens, {
          unfilteredTokens: dictionary.unfilteredTokens,
          usesDtcg,
        }),
      )
    : dictionary.allTokens;

  const result = allTokens.map(
    createFigmaToken({ dictionary, outputReferences, usesDtcg }),
  );

  const [variables, styles] = result.reduce(
    (acc, token) => {
      if (!token) return acc;

      if (token.tokenType === 'variable') {
        acc[0].push(token);
      } else {
        acc[1].push(token);
      }
      return acc;
    },
    [[], []] as [FigmaToken[], FigmaToken[]],
  );

  return {
    variables,
    styles,
  };
};

const createFigmaToken =
  ({
    outputReferences,
    dictionary,
    usesDtcg,
  }: {
    outputReferences?: OutputReferences;
    dictionary: Dictionary;
    usesDtcg?: boolean;
  }) =>
  (token: TransformedToken): FigmaToken | null => {
    let figmaToken: FigmaToken | null = null;

    const type = usesDtcg ? token.$type : token.type;
    const name = token.path.join('/');
    const description = token.$description ?? token.comment;

    let value = usesDtcg ? token.$value : token.value;
    const originalValue = usesDtcg
      ? token.original.$value
      : token.original.value;

    const shouldOutputRef =
      usesReferences(originalValue) &&
      (typeof outputReferences === 'function'
        ? outputReferences(token, { dictionary, usesDtcg })
        : outputReferences);

    /*
     * 参照を残すようにする
     *
     * transform 時点でオブジェクトの構造が変わってしまっている場合は、うまく機能しないことに注意。
     *
     * 例:
     *  val: { color: { value: "#fff" } }, originalVal: { color: { value: "{color.background.primary.value}" } }
     *  => { color: { value: "{color/background/primary/value}" } }
     *
     *  val: { bgColor: { value: "#fff" } }, originalVal: { color: "{color.background.primary.value}" }
     *  => { bgColor: { value: "#fff" } }
     */
    if (shouldOutputRef) {
      const originalIsObject =
        typeof originalValue === 'object' && originalValue !== null;

      if (originalIsObject) {
        const replaceRefsRecursive = (val: any, originalVal: any) => {
          if (typeof val !== 'object' || typeof originalVal !== 'object') {
            return;
          }

          for (const key in val) {
            if (!originalVal[key]) {
              continue;
            }

            if (typeof originalVal[key] === 'object') {
              replaceRefsRecursive(val[key], originalVal[key]);
            } else {
              if (
                typeof originalVal[key] === 'string' &&
                originalVal[key].startsWith('{')
              ) {
                val[key] = originalVal[key].replaceAll('.', '/');
              }
            }
          }
        };
        replaceRefsRecursive(value, originalValue);
      } else {
        value = originalValue.replaceAll('.', '/');
      }
    }

    if (
      type === 'color' ||
      type === 'number' ||
      type === 'dimension' ||
      type === 'fontFamily'
    ) {
      let variableType: VariableResolvedDataType = 'STRING';
      switch (type) {
        case 'color':
          variableType = 'COLOR';
          break;
        case 'number':
          variableType = 'FLOAT';
          break;
        case 'dimension':
          variableType = 'FLOAT';
          break;
        case 'fontFamily':
          variableType = 'STRING';
          break;
      }

      figmaToken = {
        tokenType: 'variable',
        type: variableType,
        name,
        $value: value,
        $description: description,
        scopes: getVariableScopes(token),
      };
    } else if (
      type === 'typography'
      // || type === 'shadow'
    ) {
      let styleType: StyleType = 'TEXT';

      switch (type) {
        case 'typography':
          styleType = 'TEXT';
          break;
        // case 'shadow':
      }

      figmaToken = {
        tokenType: 'style',
        type: styleType,
        name,
        $value: value,
        $description: description,
      };
    }

    return figmaToken;
  };

/** 変数のスコープを取得する */
function getVariableScopes(token: TransformedToken): VariableScope[] {
  const attr = token.attributes;

  if (attr?.category) {
    switch (attr.category) {
      case 'font-size':
        return ['FONT_SIZE'];
      case 'font-family':
        return ['FONT_FAMILY'];
      case 'font-weight':
        return ['FONT_WEIGHT'];
    }
  }

  return ['ALL_SCOPES'];
}

// --- Transform ---

const transformColorValue = (val: any) => {
  if (typeof val === 'string') {
    const [r, g, b, a] = parseToRgba(val);
    return { r: r / 255, g: g / 255, b: b / 255, a };
  }
  if (typeof val === 'object' && val.rgb) {
    const { r, g, b } = val.rgb;
    return {
      r: r / 255,
      g: g / 255,
      b: b / 255,
      a: val.alpha ?? val.a,
    };
  }

  return val;
};

const transformColor: ValueTransform = {
  name: 'color/figma',
  type: 'value',
  transitive: true,
  filter: (token, options) =>
    options.usesDtcg ? token.$type === 'color' : token.type === 'color',
  transform: (token, _, options) => {
    const val = options.usesDtcg ? token.$value : token.value;
    return transformColorValue(val);
  },
};

const transformFontFamily: ValueTransform = {
  name: 'fontFamily/figma',
  type: 'value',
  transitive: true,
  filter: (token, options) =>
    options.usesDtcg
      ? token.$type === 'fontFamily'
      : token.type === 'fontFamily',
  transform: (token, _, options) => {
    const val = options.usesDtcg ? token.$value : token.value;
    if (Array.isArray(val)) {
      return val[0];
    }
    return val;
  },
};

const figmaFormat: Format = {
  name: 'figma',
  format: ({ dictionary, options = {} }) => {
    const { outputReferences, usesDtcg } = options;

    return JSON.stringify(
      formattedFigmaToken({ dictionary, outputReferences, usesDtcg }),
      null,
      2,
    );
  },
};

StyleDictionary.registerTransform(transformColor);
StyleDictionary.registerTransform(transformFontFamily);

StyleDictionary.registerTransformGroup({
  name: 'figma',
  transforms: ['attribute/cti', 'color/figma', 'fontFamily/figma'],
});

StyleDictionary.registerFormat(figmaFormat);

// --- Config ---

const sd = new StyleDictionary({
  parsers: ['yaml-parser'],
  source: ['tokens.yaml'],
  platforms: {
    css: {
      transformGroup: 'css',
      transforms: ['size/px'],
      files: [
        {
          destination: 'build/variables.css',
          format: 'css/variables',
          options: { outputReferences: true },
        },
      ],
    },
    figma: {
      transformGroup: 'figma',
      files: [
        {
          destination: 'build/figma-tokens.json',
          format: 'figma',
          options: { outputReferences: true },
        },
      ],
    },
  },
});

// --- Build ---

await sd.hasInitialized;

await sd.cleanAllPlatforms();
await sd.buildAllPlatforms();
```

:::

#### 出力される JSON

Variables と Styles に分けて出力されます。
値の変換や、読み込む順番などがすでに解決されているため、あとは Figma のプラグインで順番に読み込むだけです。

```json
{
  "variables": [
    {
      "tokenType": "variable",
      "type": "COLOR",
      "name": "color/white",
      "$value": { "r": 1, "g": 1, "b": 1, "a": 1 },
      "scopes": [ "ALL_SCOPES" ]
    },
    {
      "tokenType": "variable",
      "type": "COLOR",
      "name": "color/black",
      "$value": { "r": 0, "g": 0, "b": 0, "a": 1 },
      "scopes": [ "ALL_SCOPES" ]
    },
    {
      "tokenType": "variable",
      "type": "COLOR",
      "name": "color/red/200",
      "$value": { "r": 1, "g": 0.788235294117647, "b": 0.788235294117647, "a": 1 },
      "scopes": [ "ALL_SCOPES" ]
    },
    // 省略
    {
      "tokenType": "variable",
      "type": "FLOAT",
      "name": "font-size/md",
      "$value": 16,
      "scopes": [ "FONT_SIZE" ]
    },
    {
      "tokenType": "variable",
      "type": "FLOAT",
      "name": "font-size/lg",
      "$value": 20,
      "scopes": [ "FONT_SIZE" ]
    },
    {
      "tokenType": "variable",
      "type": "FLOAT",
      "name": "font-weight/normal",
      "$value": 400,
      "scopes": [ "FONT_WEIGHT" ]
    },
    {
      "tokenType": "variable",
      "type": "FLOAT",
      "name": "font-weight/bold",
      "$value": 700,
      "scopes": [ "FONT_WEIGHT" ]
    },
    {
      "tokenType": "variable",
      "type": "STRING",
      "name": "font-family/helvetica",
      "$value": "Hiragino Sans",
      "scopes": [ "FONT_FAMILY" ]
    }
  ],
  "styles": [
    {
      "tokenType": "style",
      "type": "TEXT",
      "name": "typography/body",
      "$value": {
        "fontFamily": "{font-family/helvetica}",
        "fontSize": "{font-size/md}",
        "fontWeight": "{font-weight/normal}"
      }
    },
    {
      "tokenType": "style",
      "type": "TEXT",
      "name": "typography/heading",
      "$value": {
        "fontFamily": "{font-family/helvetica}",
        "fontSize": "{font-size/lg}",
        "fontWeight": "{font-weight/bold}"
      }
    }
  ]
}
```

### Figma プラグインの実装

ベースは[サンプル](https://github.com/figma/plugin-samples/blob/master/variables-import-export/README.md)を使用します。（プラグインの作成については詳しくないのでテキトーです）

サンプルは参照の解決や色の変換のためにやや難解な実装になっているのですが、こちらはすでに Style Dictionary で変換しているので、シンプルになります。

```javascript:code.js
console.clear();

figma.ui.onmessage = async (e) => {
  console.log("code received message", e);
  if (e.type === "IMPORT") {
    const { fileName, body } = e;
    await importJSONFile({ fileName, body });
  }
};
if (figma.command === "import") {
  figma.showUI(__uiFiles__.import, {
    width: 500,
    height: 500,
    themeColors: true,
  });
}

async function importJSONFile({ fileName, body }) {
  const json = JSON.parse(body);

  const { collection, modeId } = await createCollection(fileName);

  const variables = {};

  for (const variableSource of json.variables) {
    const variable = await createVariable(
      collection,
      modeId,
      variableSource,
      variables,
    ).catch((e) => {
      console.error("error", e);
    });
    variables[variableSource.name] = variable;
  }

  for (const styleSource of json.styles) {
    switch (styleSource.type) {
      case "TEXT":
        await createTextStyle(styleSource, variables).catch((e) => {
          console.error("error", e.message, e);
        });
        break;
      default:
        console.warn("unsupported type", styleSource.type, styleSource);
        break;
    }
  }

  figma.notify("インポートが完了しました！");
}

/** コレクションを作成する */
async function createCollection(name) {
  const localVariableCollections =
    await figma.variables.getLocalVariableCollectionsAsync();

  const currentCollection = localVariableCollections.find(
    (e) => e.name === name,
  );

  // すでに同じ名前の Variable がある場合はそれを使う（更新）。なければ新規作成。
  const collection =
    currentCollection || figma.variables.createVariableCollection(name);
  const modeId = collection.modes[0].modeId;

  return { collection, modeId };
}

/** バリアブルを作成する */
async function createVariable(collection, modeId, variableSource, variables) {
  const { name, type, $value, $description, scopes } = variableSource;

  const localVariables = await figma.variables.getLocalVariablesAsync();
  const currentVariable = localVariables.find(
    (e) => e.name === name && e.variableCollectionId === collection.id,
  );

  const variable =
    currentVariable || figma.variables.createVariable(name, collection, type);

  if ($description) {
    variable.description = $description;
  }
  if (scopes) {
    variable.scopes = scopes;
  }

  if (isAlias($value)) {
    variable.setValueForMode(modeId, {
      type: "VARIABLE_ALIAS",
      id: variables[refToKey($value)].id,
    });
  } else {
    variable.setValueForMode(modeId, $value);
  }

  return variable;
}

/** テキストスタイルを作成する */
async function createTextStyle(styleSource, variables) {
  const { name, $value, $description } = styleSource;

  const localTextStyles = await figma.getLocalTextStylesAsync();
  const currentTextStyle = localTextStyles.find((e) => e.name === name);

  const textStyle = currentTextStyle || figma.createTextStyle();
  textStyle.name = name;

  if ($description) {
    token.description = $description;
  }

  const { fontFamily, fontSize, fontWeight, lineHeight } = $value;

  if (isAlias(fontFamily)) {
    textStyle.setBoundVariable("fontFamily", variables[refToKey(fontFamily)]);
  } else {
    // INFO: fontFamily は事前に読み込みが必要なので、多分動かない
    textStyle.fontFamily = fontFamily;
  }

  if (isAlias(fontSize)) {
    textStyle.setBoundVariable("fontSize", variables[refToKey(fontSize)]);
  } else {
    textStyle.fontSize = fontSize;
  }

  if (isAlias(fontWeight)) {
    textStyle.setBoundVariable("fontWeight", variables[refToKey(fontWeight)]);
  } else {
    textStyle.fontWeight = fontWeight;
  }

  if (isAlias(lineHeight)) {
    textStyle.setBoundVariable("lineHeight", variables[refToKey(lineHeight)]);
  } else {
    textStyle.lineHeight = lineHeight;
  }

  return textStyle;
}

function isAlias(value) {
  return typeof value === "string" && value.startsWith("{");
}

function refToKey(ref) {
  return ref.slice(1, -1);
}
```

## 実際に Figma にインポートする

プラグインを読み込まる方法省略します。

プラグインを読み込んだら、以下の手順でインポートを行います

1. 「Import Variables」を実行
2. JSON を貼り付ける
3. ボタンをクリック

![インポート手順](/images/figma-style-dictionary/import.png)

成功すると、Variables と Styles が追加されているはずです。

| Variables | Styles |
| --- | --- |
| ![Variables](/images/figma-style-dictionary/figma-variables.png)  | ![Styles](/images/figma-style-dictionary/figma-styles.png) |

# まとめ

あまり細かい説明はできませんでしたが、Style Dictionary と Figma のプラガインを組み合わせて、YAML で管理されたデザイントークンを Figma にインポートする方法を紹介しました。
デザイントークンや Figma の Variables の機能はどちらも発展途上で、これからさらに改善されていく可能性が高いと思います。
それぞれが成熟したり、デザイントークンが浸透していけば、いろんなユースケースに対応したプラグインなども生まれるかもしれないです。

現状は、**自分である程度カスタマイズできるようにしておく方が、自分たちにあったシステムを作ることができて良いと思っています。**
一度理解すれば、Style Dictionary も Figma の API もそこまで複雑ではないです。


今までは、エンジニアとして Figma を「見る」ことしかしていなかったのですが、今回のデザイントークンの構築で初めて、調査のために Figmaで「作る」体験をしてみたりしました。
そこで、Variables の Scope の機能に気づいたりと学びも得ることができました。

Figma は主にデザイナーが触ることが多いと思いますが、プラグインなどエンジニアリングできるツールでもあるので、 デザイントークンに限らずデザイナーの実際の作業などの理解などを深めて、デザイン作成作業の効率化などにエンジニアとして貢献できないか考えられたら良いなと思いました。
