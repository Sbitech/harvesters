Harvesterについて
==================

Harvesterは、コンピュータビジョンアプリケーションにおける画像取得プロセスを非常に簡単にすることを目的としたPythonライブラリです。上の絵の中の農民/収穫者のように、画像データを収穫物として集め、バケツ/バッファを満たします。

Harvesterは`Apache License-2.0`_の下で自由に使用、変更、配布することができ、ソフトウェアの使用目的（個人用、社内用、商用）を気にする必要はありません。

Harvesterが行うタスク
=====================

Harvesterの主な機能は以下の通りです：

* GenTLプロデューサーを介した画像取得
* 単一のPythonスクリプトでの複数のGenTLプロデューサーの読み込み
* GenICam特徴ノードの操作

2番目の項目は、Pythonスクリプト内でさまざまな種類のトランスポート層を使用できることを意味します。各トランスポート層にはそれぞれの長所と短所があり、アプリケーションの要件に基づいて適切なものを選択する必要があります。何らかの目的で画像を取得するだけでよく、GenTLプロデューサーが何らかの方法で画像を配信します。これこそがGenTL標準の大きな利点です！

もちろん、GenTLプロデューサーだけでなく、Harvesterは直感的な方法で単一のPythonスクリプトでカメラなどの複数のGenICam準拠機器を操作する方法も提供します。

GUIが必要ですか？
================

GUIが必要ですか？Harvesterには**Harvester GUI**と呼ばれる姉妹プロジェクトがあります。興味がある場合はこちらをご覧ください：https://github.com/genicam/harvesters_gui

.. image:: https://user-images.githubusercontent.com/8652625/43035346-c84fe404-8d28-11e8-815f-2df66cbbc6d0.png
    :align: center
    :alt: Image data visualizer

質問する
========

FAQページを用意しています。おそらく、FAQを読むだけで問題が解決するかもしれません：https://github.com/genicam/harvesters/wiki/FAQ

もしFAQにあなたが直面している問題についての記事がない場合は、次のページにアクセスして、問題に関連するチケットがあるかどうかを確認してください。まだ何も言及されていない場合は、お気軽にissueチケットを作成してください。私たちがあなたを助けることができます：https://github.com/genicam/harvesters/issues

リンク
======

.. list-table::
    -
        - ドキュメント
        - https://harvesters.readthedocs.io/en/latest/
    -
        - デジタルオブジェクト識別子
        - https://zenodo.org/record/3554804#.Xd4HSi2B01I
    -
        - EMVAウェブサイト
        - https://www.emva.org/standards-technology/genicam/genicam-downloads/
    -
        - Harvester GUI
        - https://github.com/genicam/harvesters_gui
    -
        - Issueトラッカー
        - https://github.com/genicam/harvesters/issues
    -
        - PyPI
        - https://pypi.org/project/harvesters/
    -
        - ソースリポジトリ
        - https://github.com/genicam/harvesters

IPython上のHarvester
===================

次のコードブロックは、HarvesterがIPython上で実行されている様子を示しています。取得した画像はバッファのペイロードとして配信され、``ImageAcquirer``クラスの``fetch``メソッドを呼び出すことでバッファを取得できます。画像を取得したら、すぐに画像処理を開始することができます。Jupyter notebook上で実行している場合は、Matplotlibを使用して画像データを可視化することができます。このステップは、画像処理フローにおける試行内容を確認するのに役立ちます。

.. code-block:: python

    (genicam) kznr@Kazunaris-MacBook:~% ipython
    Python 3.6.6 |Anaconda, Inc.| (default, Jun 28 2018, 11:07:29)
    Type 'copyright', 'credits' or 'license' for more information
    IPython 6.5.0 -- An enhanced Interactive Python. Type '?' for help.
    
    In [1]: from harvesters.core import Harvester
    
    In [2]: import numpy as np  # This is just for a demonstration.
    
    In [3]: h = Harvester()
    
    In [4]: h.add_file('/Users/kznr/dev/genicam/bin/Maci64_x64/TLSimu.cti')
    
    In [5]: h.update()
    
    In [6]: len(h.device_info_list)
    Out[6]: 4
    
    In [7]: h.device_info_list[0]
    Out[7]: {'display_name': 'TLSimuMono (SN_InterfaceA_0)', 'id_': 'TLSimuMono', 'model': 'TLSimuMono', 'serial_number': 'SN_InterfaceA_0', 'tl_type': 'Custom', 'user_defined_name': 'Center', 'vendor': 'EMVA_D', 'version': '1.2.3'}
    
    In [8]: ia = h.create(0)
    
    In [9]: ia.remote_device.node_map.Width.value = 8
    
    In [10]: ia.remote_device.node_map.Height.value = 8
    
    In [11]: ia.remote_device.node_map.PixelFormat.value = 'Mono8'
    
    In [12]: ia.start()
    
    In [13]: with ia.fetch() as buffer:
        ...:     # Let's create an alias of the 2D image component:
        ...:     component = buffer.payload.components[0]
        ...:
        ...:     # Note that the number of components can be vary. If your
        ...:     # target remote device transmits a multi-part information, then
        ...:     # you'd get two or more components in the payload. However, now
        ...:     # we're working with a remote device that transmits only a 2D image.
        ...:     # So we manipulate only index 0 of the list object, components.
        ...:
        ...:     # Let's see the acquired data in 1D:
        ...:     _1d = component.data
        ...:     print('1D: {0}'.format(_1d))
        ...:
        ...:     # Reshape the NumPy array into a 2D array:
        ...:     _2d = component.data.reshape(
        ...:         component.height, component.width
        ...:     )
        ...:     print('2D: {0}'.format(_2d))
        ...:
        ...:     # Here are some trivial calculations:
        ...:     print(
        ...:         'AVE: {0}, MIN: {1}, MAX: {2}'.format(
        ...:             np.average(_2d), _2d.min(), _2d.max()
        ...:         )
        ...:     )
        ...:
    1D: [123 124 125 126 127 128 129 130 124 125 126 127 128 129 130 131 125 126 127 128 129 130 131 132 126 127 128 129 130 131 132 133 127 128 129 130 131 132 133 134 128 129 130 131 132 133 134 135 129 130 131 132 133 134 135 136 130 131 132 133 134 135 136 137]
    2D: [[123 124 125 126 127 128 129 130]
     [124 125 126 127 128 129 130 131]
     [125 126 127 128 129 130 131 132]
     [126 127 128 129 130 131 132 133]
     [127 128 129 130 131 132 133 134]
     [128 129 130 131 132 133 134 135]
     [129 130 131 132 133 134 135 136]
     [130 131 132 133 134 135 136 137]]
    AVE: 130.0, MIN: 123, MAX: 137
    
    In [14]: ia.stop()
    
    In [15]: ia.destroy()
    
    In [16]: h.reset()
    
    In [17]: quit
    
    (genicam) kznr@Kazunaris-MacBook:~%

用語
====

詳細について話し始める前に、このドキュメントで頻繁に登場するいくつかの重要な用語を見てみましょう。これらの用語は以下の通りです：

* *GenApi-Python Binding*：GenICam GenApiリファレンス実装と通信するPythonモジュール。
* *GenTL Producer*：Cインターフェースを持ち、物理トランスポート層に依存する技術を介してカメラと通信する方法を消費者に提供するライブラリで、詳細を消費者から隠蔽します。
* *GenTL-Python Binding*：GenTLプロデューサーと通信するPythonモジュール。
* *Harvester*：画像取得エンジン。
* *Harvester GUI*：Harvesterベースのグラフィカルユーザーインターフェース。
* *GenICam準拠デバイス*：通常はカメラです。GenICamリファレンス実装を含むことで、消費者にターゲットリモートデバイスを動的に構成/制御する方法を提供します。

次の図は、関連するモジュールの階層と関係を示しています：

.. figure:: https://user-images.githubusercontent.com/8652625/155761972-c131d638-a0cc-4c51-aa3b-752d8f3d1284.svg
    :align: center
    :alt: Module hierarchy

Harvesterの始め方
=================

Harvesterを使い始める準備はできましたか？以下のページでさらに詳しいトピックを学ぶことができます：

* `INSTALL.rst`_：Harvesterとその前提条件のインストール方法を学びます。
* `TUTORIAL.rst`_：典型的な画像取得ワークフローでHarvesterを使用する方法を学びます。

