---
title: "WordPressのAdvanced Custom Fieldsでwidgetを作る話"
date: "2020-02-25"
categories: 
  - "未分類"
tags: 
  - "php"
  - "wordpress"
---

Advanced Custom Fieldsでフィールドを拡張したものの、日付アーカイブ的なウィジェットがないと不便極まりないが探し方が悪いのかそもそもAdvanced Custom Fieldsを使ってるのが間違いなのか情報が皆無。 よって自分でなんとかすることにした。

Advanced Custom Fieldsの導入については触れない。すでに入っている前提で執筆する。

まず、管理画面からACFのフィールドグループからフィールドを作成する。 選択肢にしたかったため自分はフィールドタイプをSelectにして複数のデータを列挙した。 このとき、選択肢にHTMLを設定した。後に説明するがこのせいでとても面倒なコーディングをした。

フィールドを作成し、記事を作成したあと、記事に設定したフィールドをウィジェットとして登録する。 その時のコードはこちらを参考にさせてもらった。 [これだけ！？WordPressのWidget（ウィジェット）の作り方](https://liginc.co.jp/web/wp/112370)

実際にはテーマのディレクトリ wp-content/themes/{テーマ名} の下にwidgetとディレクトリを掘ってファイルをそこに放り込んだ。

```
<?php
class ACF_Widget extends WP_Widget{
    private $acf_field_name = ''; // 後述(1)
    private $acf_element_name = ''; //後述(2)
    /**
     * Widgetを登録する
     */
    function __construct() {
        parent::__construct(
            'acf_widget', // Base ID
            'MyWidget', // Name
            array( 'description' => 'MyWidget', ) // Args
        );
    }

    /**
     * 表側の Widget を出力する
     *
     * @param array $args      'register_sidebar'で設定した「before_title, after_title, before_widget, after_widget」が入る
     * @param array $instance  Widgetの設定項目
     */
    public function widget( $args, $instance ) {
        global $wpdb;
        $title = $instance['title'];
        echo $args['before_widget'];

        printf('<h3 class="widget-title">%s</h3>', $title);

        $select = $this->get_list();

        // 対象フィールドのPOST数を集計
        $query = "
          SELECT
            meta.meta_value AS value, COUNT(DISTINCT post.id) AS post_count
          FROM
            {$wpdb->posts} AS post
            INNER JOIN {$wpdb->postmeta} AS meta ON post.id = meta.post_id
          WHERE
            post.post_type = 'post'
            AND meta.meta_key = %s
            AND post.post_status = 'publish'
          GROUP BY
            value
        ";
        $result = $wpdb->get_results($wpdb->prepare($query, $this->acf_element_name));

        // そのままでは使いづらいので連想配列に変換
        $array_to_hash = array();
        foreach($result as $row) {
          $array_to_hash[md5($row->value)] = $row->post_count; // valueがhtmlタグも含むためmd5で不穏な文字列をなくす
        }

        echo "<ul>";
        foreach($select as $value) {
          // 選択肢(記事数) というレイアウトを作る
          $hash = openssl_encrypt($value, 'AES-256-CBC', '秘密鍵16進数文字列');
          $count = $array_to_hash[md5($value)];
          printf('<li><span>%s (<a href="/?val=%s">%d</a>)</span></li>', $value, urlencode($hash), $count);
        }
        echo "</ul>";

        echo $args['after_widget'];
    }

    /** Widget管理画面を出力する
     *
     * @param array $instance 設定項目
     * @return string|void
     */
    public function form( $instance ){
        $title = $instance['title']; // 後述(3)
        $title_name = $this->get_field_name('title');
        $esc_title = esc_attr( $title );
        echo "
        <p>
            <label for='$title_id'>title:</label>
            <input class='widefat' id='$title_id' name='$title_name' type='text' value='$esc_title'>
        </p>
        ";
    }

    /**
     * @return array ACFのSELECT要素のvalueを返す
     */
    private function get_list() {
      // フィールドグループの記事オブジェクトを取得
      $post_param = array(
        'posts_per_page' => 1,                        // 取得する数
        'post_type'      => 'acf-field',              // 記事のタイプ
        'name'           => $this->acf_field_name,    // ACFの対象フィールドの名前
      );
      $post = get_posts($post_param);
      $acf_post = array_shift($post);

      $unserialize = maybe_unserialize( $acf_post->post_content );

      return $unserialize['choices'];
    }
}

add_action( 'widgets_init', function () {
    register_widget( 'ACF_Widget' );  //WidgetをWordPressに登録する
    register_sidebar( array(  //「サイドバー」を登録する
        'name'          => 'サイドバー(上部)',
        'id'            => 'my_sidebar',
        'before_widget' => '<div>',
        'after_widget'  => '</div>',
        'before_title'  => '',
        'after_title'   => '',
    ) );
} );

```

ウィジェット部分のロジックはこんな感じである。 これを適当な名前つけてfunctions.phpからinclude\_once( 'widget/hoge.php'); とかしてロードすれば管理画面上にウィジェットが現れるはずなので配置する。

まず1の部分、 ここは

```
 SELECT post_title, post_name FROM wp_posts WHERE post_type = 'acf-field';
```

などとDBに投げればわかる対象のフィールドのwp\_postmeta.meta\_valueでfield\_xxxxxを設定する。

次に2の部分、ここは単純にACFのフィールド名を入れればいい。 フィールドを作るときに設定したはずなので値は適宜変更してほしい、

そして3の部分。このへんはリンク元のサイトを参照してもらえればわかると思うがウィジェットに設定する項目である。 このサンプルではタイトルを設定できるようにしてある。

さて、ここまではウィジェットの作り方だ。 次はこのウィジェットを元に、リンク先を用意する。 このままでは該当する属性を持つ記事数がわかるウィジェットでしかない。

ベテランエンジニアならすでにお気づきかもしれないがリンクの文字列にopenssl\_encryptを利用している。 ACFのセレクトフィールドでコロン(:)で区切るとvalueとlabelを分けられることを知らなかったためにこのような悲しい実装になっているが知っていても結局困ったと思う。

リンク先の用意は簡単だ。 下記のコードを用いればいい。

acf\_search.phpとか名前つけてこれまたfunctions.phpでinclude\_onceする。

```
<?php                                                                                                                                

function acf_search( $query ) {
    // ウィジェットやメニューでも呼ばれるのでmainクエリ以外無視する                                                                  
    if(!$query->is_main_query()) {                                                                                                   
      return;
    }
    
    $list = array('val');                                                                                           
    
    $args = $query->get( 'meta_query', [] );
                                                                                                                                     
    foreach($list as $val) {
      if(!isset($_GET[$val])) {                                                                                                      
        continue;
      }                                                                                                                              
      switch($val) {
        case 'val':
          $args[] = [                                                                                                                
              'key'     => '', // フィールド名                                                                                      
              'value'   => openssl_decrypt($_GET[$val], 'AES-256-CBC', '秘密鍵16進数文字列'), // 暗号化時と同じキー                     
              'compare' => '=',
          ]; 
          break;                                                                                                                  
        default:
          break;
      }                                                                                                                              
    }
    $query->set( 'meta_query', $args );                                                                                              
                                                                                                                                     
}

add_action( 'pre_get_posts', 'acf_search' );
```

と、pre\_get\_postsフックを使ってクエリを追加する。 ここでのvalのやり取りは別に機密情報などではないのだがｈｔｍｌタグで受け渡す関係上、可逆暗号化してセーフティな文字列にしたかっただけでわかりやすく数値や英字を利用しているなら特に不要だと思う。
